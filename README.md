# 손가락 인식을 활용한 슬라이더 및 LED 밝기 제어

웹캠으로 손가락의 움직임을 인식하고, 슬라이더 값을 변경하여 아두이노 LED의 밝기를 조절하는 프로젝트이다. p5.js와 ml5.js로 손을 인식하고, Web Serial 통신으로 아두이노에 값을 전송하였다.

## 1. 회로도

<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/deaa463c-99c4-4671-907e-982c1a425449" />


아두이노 Uno, LED, 저항, 브레드보드를 사용하였다. 아두이노의 PWM 핀을 저항과 LED에 연결하고, LED의 음극을 GND에 연결하였다.

PWM 핀 번호는 아두이노 코드의 `LED_PIN` 설정과 일치하도록 구성하였다.

## 2. 코드 설명

🔗 [p5.js 전체 코드 보기](https://editor.p5js.org/bjc97r/sketches/04Ur1M6iy)

### ① 손 인식 모델 불러오기

```javascript
function preload() {
  handPose = ml5.handPose({ flipped: true });
}
```

ml5.js의 HandPose 모델을 사용하여 손의 관절 21개를 인식하였다. 모델이 준비되기 전에 인식이 시작되는 문제를 방지하기 위해 `preload()`에서 먼저 불러오도록 하였다.

### ② 중지의 구부림 판단

```javascript
const cosine = constrain(
  (ux * vx + uy * vy) / (lengthU * lengthV),
  -1,
  1
);

const angle = Math.acos(cosine) * 180 / Math.PI;

return angle < 150;
```

중지 관절을 연결하는 두 벡터 사이의 각도를 계산하였다. 중지를 펴면 약 180도가 되고, 구부리면 각도가 작아지는 원리를 이용하였다. 각도가 150도보다 작으면 밝기 조절 모드로 판단하였다.

### ③ 손가락 간격을 슬라이더 값으로 변환

```javascript
const value = round(100 * fingerDistance / palmLength) - 15;
return constrain(value, 0, 100);
```

엄지와 검지 사이의 거리를 손바닥 길이로 나누어 0~100 범위의 값으로 변환하였다. 손바닥 길이를 기준으로 사용하여 카메라와 손 사이의 거리 변화에 따른 영향을 줄였다.

```javascript
smoothValue += 0.05 * (target - smoothValue);
slider.value(round(smoothValue));
```

목표 값과 현재 값 차이의 5%씩 이동하도록 하여 슬라이더가 부드럽게 움직이도록 하였다.

### ④ 아두이노로 값 전송 및 LED 제어

```javascript
await serial.send(String(sliderValue));
```

`serial-helper.js`를 이용하여 슬라이더 값을 문자열로 전송하였다. 연결된 상태에서 약 0.2초 간격으로 값을 보내도록 구성하였다.

```cpp
int brightness = map(value, 0, 100, 0, 255);
analogWrite(LED_PIN, brightness);
```

아두이노에서는 받은 값 0~100을 PWM 출력 범위인 0~255로 변환하여 LED 밝기를 조절하였다.

## 3. 실제 작동 모습

중지를 구부린 상태에서 엄지와 검지 사이의 간격을 변경하면 슬라이더 값과 LED 밝기가 함께 변한다.

### 최솟값: 0

![LED 꺼짐] <img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/b137a090-2570-4267-9c9f-21e65c32eb72" />


슬라이더 값이 0일 때 LED가 꺼진다.

### 중간값

![LED 중간 밝기](<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/a6e119eb-61a2-46a4-a029-6ada59653363" />
)

슬라이더를 중간값으로 조절하면 LED가 중간 밝기로 켜진다.

### 최댓값: 100

![LED 최대 밝기](<img width="4000" height="3000" alt="image" src="https://github.com/user-attachments/assets/d89b7409-1155-400f-bd11-13b266d41042" />
)

슬라이더 값이 100일 때 LED가 최대 밝기로 켜진다.

### 작동 동영상

[▶ 손가락 인식 및 LED 밝기 제어 영상](

https://github.com/user-attachments/assets/4b7f0673-db74-4770-b1f5-1ec763d356a0

)

## 4. 고찰

이번 프로젝트를 통해 손 인식 결과를 슬라이더 값으로 변환하고, 이를 실제 LED 제어로 연결하는 과정을 이해하였다.

처음에는 손 인식 모델의 로딩 시점과 중지의 구부림 판단에서 문제가 발생하였다. 모델을 먼저 불러오도록 수정하고, 중지의 구부림을 관절 각도로 판단하도록 변경하였다.

손의 방향이나 손가락이 겹치는 정도에 따라 인식이 불안정해질 수 있으므로, 향후에는 판단 기준을 보완하여 안정성을 높일 필요가 있다.
