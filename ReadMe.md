# Interface for the CLP

this code contains the interface to diagnose CL (Cleft Lip) and CLP (Cleft Lip and Palate) from normal conditions.


## 1. Environment
To activate the environment, run the following command:


Make sure you have [Conda](https://docs.conda.io/en/latest/miniconda.html) installed before proceeding.

### Python & Package Versions

- **Python**: 3.11  
- **PyTorch**: 1.12.0  
- **ONNX Runtime**: 1.8.1  


### 1.1 **Create and activate the Conda environment:**

    ```bash
    conda create -n clp python=3.11 -y
    conda activate clp
    conda install pip -y
    pip install -r requirements.txt
    ```



## 2. main inferface Function
The main interface function is provided in dclp.py.


## 3. Tutorial
Here's a tutorial on how to use the interface:

```python
from dclp_tool import DCLP
import glob
dclp=DCLP() 
p_images=glob.glob("/home/weki/clp/alfyyy-yc-bqx-24w/*.png")

result=dclp.detect_and_classify(p_images)
print(result["Diagnosis"]) # the diagnosis information
```
The result is a dictionary that contains:

-  "Diagnosis"
    A dictionary that indicates the diagnosis.

-  "Support"
     the classification results of all images


-  "Det_imgs"
    - The images plotted with anatomy structures.
    - Normal anatomy is indicated with a green box.
    - Abnormal anatomy is indicated with a red box.


## 4. clinical information

### 切面类型

| 中文名称       | 英文名称                        | 缩写  |
| -------------- | ------------------------------- | ----- |
| 正常鼻唇冠状切面 | Normal Lip View                | NLV   |
| 正常上牙槽切面 | Normal Alveolar and Palate View | NAPV  |
| 异常鼻唇切面   | Cleft Lip View                  | CLV   |
| 异常上牙槽切面 | Cleft Alveolar and Palate View  | CAPV  |

### 解剖结构

| 中文名称 | 英文名称   | 缩写 |
| -------- | ---------- | ---- |
| 上唇     | Upper Lip  | UL   |
| 鼻子     | Nose       | N    |
| 鼻孔     | Nostrils   | No   |
| 下唇     | Lower Lip  | LL   |
| 下巴     | Chin       | C    |
| 牙槽     | Alveolus   | A    |

### 异常解剖结构

| 中文名称 | 英文名称     | 缩写 |
| -------- | ------------ | ---- |
| 异常上唇 | Cleft Lip    | CL   |
| 异常上牙槽 | Cleft Alveolus | CA   |
| 异常继发腭 | Cleft Palate   | CP   |

## 5. Map informaiton

### 1.   Mapping of classification:

|Code	|Description|
|---|---|
|0|	Normal Alveolar and Palate View (NAPV)|
|1|	Normal Lip View (NLV)|
|2|	Cleft Lip View (CLV)|
|3|	Cleft Alveolar and Palate View (CAPV)|



### 2.  Here is the detailed mapping for normal and abnormal anatomy structures:


### Normal Anatomy
|Code|	Description|
|---|---|
|61|	Nostrils (No)|
|62|	Upper lip (UL)|
|63|	Nose (N)|
|64|	Lower lip (LL)|
|65|	Chin (C)|


### Abnormal Anatomy
|Code|	Description|
|---|---|
|163|	Cleft alveolar (CA)|
|164|	Cleft lip (CL)|
|166|	Cleft palate (CP)|
|167|	CL_ROI|

## 6. Double List
This list contains codes that may appear more than once:

```python
double_list = [61]
```

## 7. Diagnosis Logic
The following code snippet demonstrates the logic used to determine the diagnosis based on the classification results:
```python
def diagnose(cls_set, cls_list):
    p_result = {}
    
    exists_BC = any(element for element in cls_set if element in set(["NLV", "CLV"]))
    exists_SYC = any(element for element in cls_set if element in set(["NAPV", "CAPV"]))

    if exists_BC and exists_SYC:
        if "CLV" in cls_list and "CAPV" in cls_list:
            p_result["Diagnosis"] = {"Abnormal": "CLP"}
        elif "CLV" not in cls_list and "CAPV" in cls_list:
            p_result["Diagnosis"] = {"Normal": "Normal"}
        elif "CLV" in cls_list and "CAPV" not in cls_list:
            p_result["Diagnosis"] = {"Abnormal": "CL"}
        elif "CLV" not in cls_list and "CAPV" not in cls_list:
            p_result["Diagnosis"] = {"Normal": "Normal"}
    elif exists_BC and not exists_SYC:
        p_result["Diagnosis"] = {"Uncertain": "without alveolar&palate view"}
    elif not exists_BC and exists_SYC:
        p_result["Diagnosis"] = {"Uncertain": "without lip view"}
    elif not exists_BC and not exists_SYC:
        p_result["Diagnosis"] = {"invalid": "no lip view and no alveolar&palate view"}

    return p_result
```

### Diagnosis Logic Table
|Condition|	Diagnosis|
|---|---|
|exists_BC and exists_SYC| Valid input|
|exists_BC and not exists_SYC|	Uncertain: without alveolar&palate view|
|not exists_BC and exists_SYC|	Uncertain: without lip view|
|not exists_BC and not exists_SYC|	Invalid: no lip view and no alveolar&palate view|


Valid input diagnosis

|Condition|	Diagnosis|
|---|---|
| If "CLV" in cls_list and "CAPV" in cls_list:| Abnormal: CLP|
| If "CLV" not in cls_list and "CAPV" in cls_list:| Normal: Normal|
| If "CLV" in cls_list and "CAPV" not in cls_list:| Abnormal: CL|
| If "CLV" not in cls_list and "CAPV" not in cls_list:| Normal: Normal|



## 8. Models Download

[google drive link](https://drive.google.com/drive/folders/1813fxUThopEOnOtaotjylm8zgK9S1ENE?usp=drive_link)