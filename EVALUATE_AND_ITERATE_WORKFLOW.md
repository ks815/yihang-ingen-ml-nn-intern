# AURA "Evaluate-and-Iterate" Workflow —工作原理总结

> 来源：本仓库 [ESP32/main.py](ESP32/main.py)、[model_training/server.py](model_training/server.py)、[model_training/data/](model_training/data/)（实际代码 + 数据反推得出，不是文档里现成写好的说法）。
> 用途：作为 InGen Dynamics 实习 Week 2（"Edge-Constrained Neural Network Classifier"）任务的参考基线——该任务明确要求"following the same evaluate-and-iterate workflow used in AURA"。

## 1. 整体角色分工

AURA 是一个"边采集、边训练、边推理"的闭环系统，由两端组成：

| 组件 | 角色 | 关键文件 |
|---|---|---|
| ESP32 固件 | 传感器采样、手动模式标注、构建训练窗口、发起推理请求 | [ESP32/main.py](ESP32/main.py) |
| Flask 后端 | 保存带标签数据、训练分类器、提供推理接口 | [model_training/server.py](model_training/server.py) |

模型是 `sklearn.neural_network.MLPClassifier`（隐藏层 32→16），标签为四种模式：`STUDY / RELAX / SLEEP / AWAY`。

## 2. 闭环流程（Collect → Label → Train → Infer → Iterate）

### 2.1 采样（Collect）— `ESP32/main.py`

ESP32 每 `SAMPLE_INTERVAL_MS`（1.75s）采集一次传感器读数（lux×2、noise×2、motion、lux_diff、noise_diff，共 7 维特征），滚动累积进 `sensor_buffer`，窗口大小 `WINDOW_SIZE = 30`：

```python
# ----- Configuration -----
SAMPLE_INTERVAL_MS = 1750
WINDOW_SIZE = 30          # Bigger window for ML stability
ML_PREDICTION_INTERVAL = 60   # Only classify every _ seconds

sensor_buffer = []

# --- 5. CLOUD SYNC (build ML window) ---
if time.ticks_diff(time.ticks_ms(), last_cloud_update) > SAMPLE_INTERVAL_MS:

    # 1. Append one sensor reading
    sensor_buffer.append(sample)

    # Trim BEFORE printing
    if len(sensor_buffer) > WINDOW_SIZE:
        sensor_buffer.pop(0)

    print("Collected sample:", len(sensor_buffer), "/", WINDOW_SIZE)

    # 3. If manual mode AND window full → send to server
    if manual_override and len(sensor_buffer) == WINDOW_SIZE:
        send_status_to_server(current_mode)

    last_cloud_update = time.ticks_ms()
```

### 2.2 人工打标（Label）— `ESP32/main.py`

当用户手动选择模式（按键触发 `manual_override = True`）且缓冲区凑满 30 帧时，把这 30×7=210 维向量连同当前手动选定的模式标签一起 POST 到后端 `/status` 接口。**真值标签来自用户的手动操作**，不是自动标注：

```python
def send_status_to_server(mode):
    """
    Sends the REAL buffered 10-sample training window to the server (/status)
    """
    global sensor_buffer

    try:
        if len(sensor_buffer) < WINDOW_SIZE:
            print("Not enough samples yet to send training data.")
            return

        payload = {
            "mode": mode.upper(),
            "data": sensor_buffer[:]     # full real 10-sample buffer
        }

        res = urequests.post(f"{SERVER_URL}/status", json=payload)
        print("Status send result:", res.text)
        res.close()

        # Clear buffer once successfully sent
        sensor_buffer = []

    except Exception as e:
        print("Failed to send /status:", e)
```

### 2.3 入库（Persist）— `model_training/server.py`

后端 `/status` 把新样本追加进 `data/X.npy` / `data/y.npy`。目前仓库里已经积累了 **459 条样本**，标签分布很不均衡：`SLEEP=240, RELAX=158, STUDY=39, AWAY=22`：

```python
DATA_X = "data/X.npy"
DATA_Y = "data/y.npy"
MODEL_FILE = "model/model.pkl"

def load_dataset():
    if os.path.exists(DATA_X) and os.path.exists(DATA_Y):
        X = np.load(DATA_X)
        y = np.load(DATA_Y, allow_pickle=True)   # ← Fix
        return X, y
    return np.zeros((0, 210)), np.array([], dtype=str)   # 30 samples × 7 features = 210

def save_dataset(X, y):
    np.save(DATA_X, X)
    np.save(DATA_Y, y)

# ============================
#   /status — save training data
# ============================

@app.post("/status")
def status():
    content = request.json
    mode = content["mode"]           # "study_mode"
    data = content["data"]           # 30 × [int(lux1), int(lux2), int(n1), int(n2), int(motion), int(lux_diff), int(noise_diff)]

    # Flatten into 210-length vector
    flat = np.array(data).flatten()  # shape (210,)

    if flat.shape[0] != 210:
        return jsonify({"error": f"Expected 210 features, got {flat.shape[0]}"}), 400

    # Load old dataset
    X, y = load_dataset()

    # Add new example
    X = np.vstack([X, flat])
    y = np.hstack([y, mode])

    save_dataset(X, y)

    return jsonify({"message": "Saved", "count": len(y)})
```

### 2.4 训练（Train）— `model_training/server.py`

样本数 ≥30 时，调用 `/train` 接口对**全部累积数据**重新 `model.fit(X, y)`，覆盖保存 `model/model.pkl`。**注意：这一步没有 train/test split，也没有计算 accuracy / precision / recall / F1** ——当前实现里只有训练，没有真正的"评估"代码：

```python
# ============================
#   /train — trains NN
# ============================

@app.post("/train")
def train():
    X, y = load_dataset()

    if len(y) < 30:
        return jsonify({"error": "Not enough training samples"}), 400

    print("Training model with", len(X), "samples...")

    model = MLPClassifier(
        hidden_layer_sizes=(32, 16),
        max_iter=500
    )

    model.fit(X, y)
    save_model(model)

    return jsonify({"message": "Model trained"})
```

### 2.5 推理（Infer）— ESP32 端发起，后端 `server.py` 响应

非手动模式下，固件每隔 `ML_PREDICTION_INTERVAL`（60s）把当前窗口 POST 到 `/get-mode`，取模型预测结果作为自动模式：

```python
# ESP32/main.py
def get_auto_mode():
    global sensor_buffer, last_ml_check, current_mode, manual_override

    # Do not override manual mode
    if manual_override:
        return current_mode

    # Not enough data
    if len(sensor_buffer) < WINDOW_SIZE:
        return current_mode

    # Don't classify too often
    if time.ticks_diff(time.ticks_ms(), last_ml_check) < ML_PREDICTION_INTERVAL * 1000:
        return current_mode

    last_ml_check = time.ticks_ms()

    try:
        payload = {"data": sensor_buffer}
        res = urequests.post(f"{SERVER_URL}/get-mode", json=payload)
        response_json = res.json()
        predicted = response_json.get("mode", "").upper()
        res.close()

        print("ML predicted:", predicted)

        if predicted not in ("STUDY", "RELAX", "SLEEP", "AWAY"):
            print("Invalid ML mode, ignoring.")
            return current_mode

        sensor_buffer = []

        return predicted

    except Exception as e:
        print("ML prediction error:", e)
        return current_mode
```

```python
# model_training/server.py
# ============================
#   /get-mode — inference
# ============================

@app.post("/get-mode")
def get_mode():
    model = load_model()
    if model is None:
        return jsonify({"mode": "study"})

    data = request.json["data"]  # 30 samples
    flat = np.array(data).flatten().reshape(1, -1)

    prediction = model.predict(flat)[0]
    return jsonify({"mode": prediction})
```

### 2.6 迭代（Iterate）

用户继续手动切换模式 → 产生新标注样本 → 再次调用 `/train` → 模型随数据增长持续更新。这个"用户手动纠偏 = 打标签 → 重训 → 再推理"的循环，就是所谓的 evaluate-and-iterate（更准确地说是 **collect-and-retrain** 循环，因为目前没有量化的"evaluate"步骤）。

## 3. 与实习计划文档中说法的差异（重要）

实习计划 PDF 里提到 AURA"84.91% accuracy"、"following the same evaluate-and-iterate workflow"，但**这个准确率数字和正式的评估代码（train/test split、precision/recall/F1）在当前仓库里并不存在**——现有代码只做了"全量数据训练 + 直接上线推理"，没有留出 held-out 测试集，也没有输出分类指标。84.91% 很可能是之前某次线下/notebook 里单独跑出来的结果，不在这份代码里。

**这对 Week 2 任务的实际含义**：不能只是照抄现有 `server.py` 的训练逻辑，而是要把这个"手动打标 → 全量训练"的操作流程，**升级成一个有正式评估的 ML pipeline**：
- 划分 held-out test split（且不能用于训练或调参）
- 用测试集算 per-class accuracy / precision / recall / F1
- 加上量化（INT8）和延迟基准测试
- 这些正是 Week 2 deliverable（`W02_Edge_Classifier.ipynb`、`W02_Edge_Deploy_Memo.md`）明确要求的内容。

## 4. Week 2 任务可以直接复用的部分

- **特征工程思路**：多传感器窗口化（时间序列切窗 → 展平成固定长度向量）→ AURA 的 30 帧×7 特征 = 210 维，可以类比映射到 Aido Rover 的 IMU / motor current / proximity 传感器窗口。
- **模式/状态标签体系**：AURA 的 `STUDY/RELAX/SLEEP/AWAY` 对应 Week 2 要求里提到的 `PATROL/ALERT/CHARGING/FAULT`。
- **轻量模型选型**：AURA 用小型 MLP（32,16）；Week 2 要求换成 PyTorch 的小型 feedforward 或 1D-CNN，并加上量化，这是同一设计哲学（边缘可部署、参数量小）在新框架下的延伸。
- **需要新增、AURA 当前没有的部分**：held-out test split、per-class 指标计算、量化前后延迟对比（100+ warm-up-free runs）。
