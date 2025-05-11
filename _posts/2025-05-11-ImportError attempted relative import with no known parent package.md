---
layout: single
title: "[Error] ImportError: attempted relative import with no known parent package"
published: true
categories:
  - PythonError
tag:
  - Error
  - Python
---

모듈 테스트 중 생긴 오류다.

## 오류 원인

`from .config import SATURATION_TOLERANCE`와 같이 `.`로 시작하는 임포트 구문을 상대 임포트라고 한다.

상대 임포트는 현재 모듈이 속한 패키지를 기준으로 다른 모듈을 임포트하는 것으로, 해당 오류는 상대 임포트를 잘 못 설정하여 파일을 인식할 수 없다는 오류다.

즉, 모듈 테스트 중에는 해당 모듈만 python이 인식하도록 올리는데, `from .config import …`와 같은 상대 임포트가 나타나면, config는 같이 올리지 않았으니 같은 경로에 있다고 해도 python이 인식하지 못한다는 뜻

## 해결 방법

`if __name__ == "__name__"` 블록은 해당 스크립트를 직접 실행했을 때 테스트 코드가 동작하도록 하기 위한 것으로, 이 블록 안에서 상대 임포트 오류 없이 모듈을 테스트하려면, 스크립트가 직접 실행될 때 실행되는 python 파일의 폴더를 패키지로 인식하거나, 필요한 모듈을 절대 경로로 찾을 수 있도록 `sys.path`를 임시로 수정하는 방법이 사용된다.

### 문제 소스

```python
import cv2
import numpy as np
import os

from .config import SATURATION_TOLERANCE
from .config import REFERENCE_IMAGE_RELATIVE_PATH

# 정상 이미지의 평균 채도
reference_avg_saturation = None

def calculate_average_saturation(image):
    """
    OpenCV 이미지를 받아 평균 채도를 계산한다.
    """
    
    if image is None:
        return None
    try:
        hsv_img = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
        _, saturation, _ = cv2.split(hsv_img)
    except Exception as e:
        print(f"평균 채도 계산 중 오류 발생: {e}")
        return None
    
if __name__ == "__main__":
    print("--- image_processor.py 테스트 시작 ---")
    img_path = REFERENCE_IMAGE_RELATIVE_PATH
    print(img_path)
    print("--- image_processor.py 테스트 끝 ---")
```

### 수정 소스
```python
import cv2
import numpy as np
import os
import sys

try:
    from src.config import SATURATION_TOLERANCE
    from src.config import REFERENCE_IMAGE_RELATIVE_PATH
except ImportError:
    print("상대 경로 임포트 실패.")
    try:
        current_dir = os.path.dirname(os.path.abspath(__file__)) # src 폴더 절대 경로
        project_root = os.path.dirname(current_dir) # 프로젝트 루트 절대 경로

        if project_root not in sys.path:
             sys.path.insert(0, project_root)
             print(f"프로젝트 루트 '{project_root}'를 sys.path에 추가했습니다.")
        else:
             print(f"프로젝트 루트 '{project_root}'는 이미 sys.path에 있습니다.")

        from src.config import SATURATION_TOLERANCE
        from src.config import REFERENCE_IMAGE_RELATIVE_PATH
        print("config 모듈을 절대 경로로 임포트했습니다.")

    except ImportError as e:
        print(f"FATAL ERROR: config 모듈을 임포트할 수 없습니다. 경로 또는 파일 존재 여부를 확인하세요: {e}")
        sys.exit(1) # 오류 코드와 함께 프로그램 종료

# 정상 이미지의 평균 채도
reference_avg_saturation = None

def calculate_average_saturation(image):
    """
    OpenCV 이미지를 받아 평균 채도를 계산한다.
    """
    
    if image is None:
        return None
    try:
        hsv_img = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
        _, saturation, _ = cv2.split(hsv_img)
    except Exception as e:
        print(f"평균 채도 계산 중 오류 발생: {e}")
        return None
    
if __name__ == "__main__":
    current_dir_main = os.path.dirname(os.path.abspath(__file__)) # src 폴더 절대 경로
    project_root_main = os.path.dirname(current_dir_main) # 프로젝트 루트 절대 경로

    print(f"current_dir: {current_dir_main}")
    print(f"project_root: {project_root_main}")
    print(f"sys.path: {sys.path}")
```