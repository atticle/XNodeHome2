# 실습장치와 PC 사이 통신
```sh
            |------<송신 버퍼>-----   -------<수신 버퍼>-----|  
<실습장치><포트>                     X                     <포트><PC>  
            |------<수신 버퍼>-----   -------<송신 버퍼>-----|  
```

## 실습장치에서 시리얼 통신
실습장치에서 시리얼 포트로 데이터를 보내고 받을 때는 표준함수인 input(), print() 또는 pop 라이브러리에서 제공하는 Uart.read()/Uart.readLine(), Uart.write() 사용

### 파이썬 표준 입출력
- print()는 문자열 출력 결과를 표준 출력 버퍼(STDOUT)에 전달
  - STDOUT은 운영체제에서 관리
  - 운영체제는 STDOUT에 저장된 문자열을 그래픽 어댑터의 텍스트 버퍼에 전달
  - 그래픽 어댑터는 텍스트 버퍼 내용을 명령 프롬프트 같은 텍스트 출력창에 표시
- input()는 표준 입력 버퍼(STDOUT)에 마지막에 줄바꿈 문자(<ENTER> 문자)가 포함된 문자열을 읽음
  - STDIN은 운영체제에서 관리
  - 운영체제는 키보드가 <ENTER>를 입력할 때까지 앞서 입력한 문자들을 STDIN에 저장

### 마이크로파이썬의 print()와 input()
- 마이크로파이썬 인터프리터는 STDOUT을 시리얼의 출력 버퍼에 전달
  - print()는 시리얼 송신
- 마이크로파이썬 인터프리터는 시리얼 입력 버펀을 STDIN에 전달
  - input()은 시리얼 수신

### pop 라이브리리와 시리얼 통신
- Uart 객체를 통해 시리얼 송수신 지원. 문자열은 내부에서 인코딩/디코딩 처리
  - write(): 문자열을 시리얼로 전송
  - read(): 시리얼로부터 일정 크기의 문자열 수신. 수신된 문자열이 읽을 크기보다 작으면 모두 읽을 때까지 대기함
  - readLine(): 시리얼로부터 줄단위 문자열 수신. 줄단위 문자가 수신될 때까지 대기함

## PC에서 시리얼 통신
PC의 시리얼 포트에서 파이썬으로 데이터를 읽고 쓰려면 pyserial 라이브러리 사용하며, replx 툴을 설치하면 함께 설치됨 

### pyserial

다음 명령으로 PC에 pyserial이 설치되었는지 확인
```sh
pip list | findstr "pyserial"
```

만약 설치되어 있지 않다면 다음과 같이 설치
```sh
pip install pyserial
```

pyserial이 Serial 객체로 PC이 시리얼 포트에서 데이터를 읽고 쓸 수 있음
- Serial(port, baudrate): 시리얼 객체 생성
  - port: 사용할 포트 이름. 실습장치가 연결된 포트 사용
  - baudrate: 전송속도. **115200**으로 설정
- in_waiting: 프로퍼티로 현재 수신 버퍼에 저장된 바이트 문자열 크기
- out_waiting: 프로퍼티로 현재 송신 버퍼에 저장된 바이트 문자열 크기
- write(): 바이트 문자열을 시리얼로 전송. 문자열은 전송 전에 반드시 바이트 문자열로 변환해야 함
  - 대량의 데이터를 전송할 때는 out_waiting으로 송신 버퍼의 여유 크기를 확인하면서 전송.
- flush() 송신 버퍼의 내용을 강제로 전송함
- read(): 시리얼로부터 일정 크기의 바이트 문자열 수신. 수신된 바이트 문자열이 읽을 크기보다 작으면 모드 읽을 때까지 대기함
  - 현재 수신된 모든 데이터를 읽을 때는 in_waiting을 인자로 사용
- readline(): 시리얼로부터 줄단위 문자열 수신. 줄단위 바이트 문자가 수신될 때까지 대기함
- reset_input_buffer(): 수신 버퍼를 모두 비움
- reset_output_buffer(): 송신 버퍼를 모두 비움
- close(): 통신 종료

### 에코 테스트
PC에서 전송한 문자열을 실습장비가 다시 PC로 전송함으로써 정상적으로 통신이 수행됨을 확인

1. PC에서 실습장비를 위한 새 파일(serial_xnode.py)을 만든 후 다음 코드 입력
```python
from xnode import ReplSerial

serial = ReplSerial(0.1)

def setup():
    pass

def loop():
    cmd = serial.read_until(b"\r\n")
    if cmd:
        serial.write(cmd)
        
if __name__ == "__main__":
    setup()
    while True:
        loop()
```

- 마이크로파이썬의 input(), print() 대신 xnode 라이브러리의 ReplSerial.read_until()과 ReplSerial.write() 권장
  - input(), print()는 PC에서 pyserial로 데이터를 주고 받을 때 추가 처리 필요
    
2. replx 툴의 run 명령으로 작성할 파일을 실행한 후 오류가 없는지 결과 확인
```sh
replx com4 run --text serial_xnode.py
```
- Ctrl+T를 여러 차례 눌러 구분자를 <CR><LF>로 변경 후 문자열을 입력 및 \<Enter>를 누르면, 실습장비에서 수신한 문자열을 다시 PC로 전송함
- 오류가 없으면 \<Ctrl>+c를 눌러 replx 툴 종료 

3. replx 툴을 강제로 종료한 후 다시 run -n 명령으로 작성한 파일을 실습장비에 실행 및 replx 툴 종료
```sh
replx com4 run -n serial_xnode.py
```

4. PC를 위한 새 파일(serial_pc.py)을 만든 후 다음 코드 입력
```python
from serial import Serial 

ser = Serial("COM4", 115_200)

while True:
    data = input(">")
    ser.write((data + '\n').encode())
    ret = ser.readline().decode()
    print(ret)
```

5. 작성한 파일을 python 인터프리터로 PC에서 실행
```sh
python serial_pc.py
```
- 문자열을 입력한 후 \<Enter>를 누르면 실습장비로 문자열이 전송되고, 실습장비에서 다시 PC 보낸 문자열을 읽어 화면에 표시

## 시리얼 통신 응용
실습장비의 확장 포트에 IMU 확장을 꼽은 후 PC에서 시리얼 통신으로 실습장비의 가속도 센서값을 읽은 후 필요한 응용 구현  
관성 센서는 가속도(3축), 각속도(3축), 지자기(3축)으로 구성되며, 자동차, 드론 등에서 충격 감지, 자세 제어 등에 활용 
- 가속도(액설러미터): 해당 축 방향에 가해진 순간 충격 또는 기울기
- 각속도(자이로스코프): 해당 축 방향으로 순간 회전한 각도 
- 지자기(마그네틱): 해당 축 방향의 자성 세기

축은 직선 방향을 나타내며 XNode 모트를 바닦에 놓은 상태에서 x, y 축은 IMU 확장에 인쇄된 그램으로 확인, z축은 하늘과 땅을 잇는 선  

### 실습장비의 IMU 센서값을 PC에서 읽기

**실습장비(imu_xnode.py)**
```python
from xnode import ReplSerial
from xnode.imu import IMU
from time import sleep_ms

serial = ReplSerial()
imu = IMU() 

while True:
    x, y, z = imu.read(IMU.ACCELERATION)
    serial.write(b"%6.2f %6.2f %6.2f\r\n" % (x, y, z)) 
    sleep_ms(20) 
```

**PC(imu_pc.py)**
```python
from serial import Serial, SerialException
from time import sleep

PORT = "COM8"
BAUDRATE = 115200

try:
    # timeout 추가로 무한 대기 방지
    with Serial(PORT, BAUDRATE, timeout=1) as ser:
        # 시작 전 기존 버퍼에 쌓인 찌꺼기 데이터 비우기
        ser.reset_input_buffer()
        old_state = None
        
        while True:
            # 버퍼에 읽을 데이터가 있을 때만 처리 (불필요한 대기 방지)
            if ser.in_waiting > 0:
                try:
                    # errors='ignore'로 깨진 바이트 무시, strip()으로 앞뒤 공백 및 줄바꿈 제거
                    ret = ser.readline().decode('utf-8', errors='ignore').strip()
                    
                    # 빈 문자열이 들어온 경우 건너뛰기
                    if not ret:
                        continue
                        
                    # 공백 기준으로 분리 후 첫 번째 값을 실수로 변환
                    x = abs(float(ret.split()[0]))
                    
                    if x >= 10.0:
                        if old_state != "large":
                            print("large")
                            old_state = "large"
                    elif x >= 6.0:
                        if old_state != "medium":
                            print("medium")
                            old_state = "medium"
                    elif x >= 2.0:
                        if old_state != "small":
                            print("small")
                            old_state = "small"
                    else:
                        if old_state != "very small":
                            print("very small")
                            old_state = "very small"
                        
                except (ValueError, IndexError):
                    # 숫자로 변환할 수 없는 이상한 데이터가 들어온 경우 무시하고 계속 진행
                    print(f"[경고] 잘못된 데이터 형식: {ret}")
                    continue
            else:
                # 데이터가 없을 때만 CPU 점유율을 낮추기 위해 짧게 대기
                sleep(0.005)

except SerialException as e:
    print(f"시리얼 포트 연결 오류: {e}")
except KeyboardInterrupt:
    print("\n프로그램을 종료합니다.")
```

### 프로세싱 구조 적용
아두이노에서 채택한 프로세싱 구조는 미디어아트를 위한 프로그래밍 방식 중 하나로 이미지를 출력하는 draw() 대신 범용 무한루 프인 loop() 사용
  
**실습장비(imu_processing_xnode.py)**
```python
from xnode import ReplSerial
from xnode.imu import IMU
from time import sleep_ms

serial = None
imu = None

def setup():
    global serial, imu
    serial = ReplSerial()
    imu = IMU()

def loop():
    x, y, z = imu.read(IMU.ACCELERATION)
    serial.write(b"%0.2f %0.2f %0.2f\r\n" % (x, y, z)) 
    sleep_ms(20) 


if __name__ == "__main__":
    setup()
    while True:
        loop()
```

**PC(imu_processing_pc.py)**
```python
from serial import Serial, SerialException
from time import sleep

PORT = "COM8"
BAUDRATE = 115200

ser = None
old_state = None

def setup():
    global ser
    try:
        ser = Serial(PORT, BAUDRATE, timeout=1)
        ser.reset_input_buffer()
    except SerialException as e:
        print(f"시리얼 포트 연결 오류: {e}")

def loop():
    global old_state
    
    if ser.in_waiting > 0:
        try:
            ret = ser.readline().decode('utf-8', errors='ignore').strip()
            
            if not ret:
                return
                
            x = abs(float(ret.split()[0]))
            
            if x >= 10.0:
                if old_state != "large":
                    print("large")
                    old_state = "large"
            elif x >= 6.0:
                if old_state != "medium":
                    print("medium")
                    old_state = "medium"
            elif x >= 2.0:
                if old_state != "small":
                    print("small")
                    old_state = "small"
            else:
                if old_state != "very small":
                    print("very small")
                    old_state = "very small"
                
        except (ValueError, IndexError):
            print(f"[경고] 잘못된 데이터 형식: {ret}")
    else:
        sleep(0.005)    

if __name__ == "__main__":
    setup()
    try:
        while True:
            loop()
    except KeyboardInterrupt:
        print("\n프로그램을 종료합니다.")
        if ser is not None:
            ser.close()
```

### 클래스로 확장
가속도 값의 크기에 따라 PC에서 해당 음악 재생  

**실습장비(imu_class_xnode.py)**
```python
from xnode import ReplSerial
from xnode.imu import IMU
from time import sleep_ms

class Processing:
    def __init__(self):
        self.setup()
        while True:
            self.loop()
        
    def setup(self):
        self.serial = ReplSerial()
        self.imu = IMU()

    def loop(self):
        x, y, z = self.imu.read(IMU.ACCELERATION)
        self.serial.write(b"%6.2f %6.2f %6.2f\r\n" % (x, y, z)) 
        sleep_ms(20) 


if __name__ == "__main__":
    Processing()
```

**PC(music_download.py)**
- ffmpeg 툴과 yt-dlp와 miniaudio 패키지 설치 

```sh
winget install Gyan.FFmpeg -s winget

pip install yt-dlp
pip install miniaudio
```

- 재생할 MP3 오디오 파일 수집

```python
import yt_dlp

# 다운로드할 목표 파일명 리스트 (검색어로도 활용)
song_list = [
    "beatles_yellow_submarine",
    "led_zeppelin_stairway_to_heaven",
    "acdc_thunderstruck",
]

def download_audio(track_names):
    for name in track_names:
        # 파일명의 '_'를 공백으로 변환하여 검색어 생성
        search_query = f"ytsearch1:{name.replace('_', ' ')} official audio"
        
        ydl_opts = {
            # 최고 음질의 오디오 스트림 선택
            'format': 'bestaudio/best',
            # 저장될 파일 경로 및 이름 지정 (확장자는 후처리기에서 자동 부여)
            'outtmpl': f'{name}.%(ext)s',
            # 플레이리스트가 검색될 경우 단일 영상만 다운로드
            'noplaylist': True,
            # FFmpeg를 이용해 MP3(192kbps)로 추출
            'postprocessors': [{
                'key': 'FFmpegExtractAudio',
                'preferredcodec': 'mp3',
                'preferredquality': '192',
            }],
        }

        print(f"[다운로드 시작] {name}.mp3 검색 중...")
        try:
            with yt_dlp.YoutubeDL(ydl_opts) as ydl:
                ydl.download([search_query])
            print(f"[완료] {name}.mp3 저장 성공\n")
        except Exception as e:
            print(f"[오류] {name} 다운로드 실패: {e}\n")

if __name__ == "__main__":
    download_audio(song_list)
```

**PC(imu_class_pc.py)**
가속도 세기에 따른 MP3 오디오 파일 재생  
- 클래스 상속을 통한 오버라이딩 기법 적용

```python
import sys
import time
import math
import signal
from pathlib import Path
from collections import deque

import miniaudio
from serial import Serial, SerialException


class Processing:
    def __init__(self):
        signal.signal(signal.SIGINT, self.cleanup)
        self.setup()
        while True:
            self.loop()

    def setup(self):
        print("call setup")

    def loop(self):
        print("call loop")
        time.sleep(1)

    def cleanup(self, *args):
        sys.exit(0)


class SimpleMp3Player(Processing):
    BASE_DIR = Path(__file__).resolve().parent

    # 4단계 흔들기 강도 임계값 (단위: m/s^2)
    TH_WEAK = 3.5      # 1단계: 3.5 ~ 11.9 (1번 곡)
    TH_MEDIUM = 12.0   # 2단계: 12.0 ~ 18.4 (2번 곡)
    TH_STRONG = 18.5   # 3단계: 18.5 ~ 25.9 (3번 곡)
    TH_STOP = 26.0     # 4단계: 26.0 이상 초강력 흔들기 (음악 정지)
    SETTLE_SEC = 0.4   # 손을 멈췄다고 판정할 대기 시간 (초)

    SAMPLE_RATE = 44100
    CHANNELS = 2
    SAMPLE_FORMAT = miniaudio.SampleFormat.SIGNED16

    def __init__(self, port="com8", baudrate=115200):
        self.port = port
        self.baudrate = baudrate
        self.ser = None
        self.device = None
        self._active_stream = None
        super().__init__()

    def setup(self):
        try:
            self.ser = Serial(self.port, self.baudrate, timeout=1)
            self.ser.reset_input_buffer()
        except SerialException as e:
            print(f"시리얼 포트 연결 오류: {e}")
            sys.exit(1)

        self.device = miniaudio.PlaybackDevice(
            output_format=self.SAMPLE_FORMAT,
            nchannels=self.CHANNELS,
            sample_rate=self.SAMPLE_RATE,
        )

        self.confirmed_flags = None         # 현재 재생 중인 곡 단계 (1, 2, 3 또는 정지 시 None)
        self.window = deque(maxlen=15)      # 최근 15개 데이터(약 0.3초) 이동 평균 버퍼
        self.peak_level = 0                 # 한 번 흔드는 동안 도달한 최고 강도 단계 (1~4)
        self.peak_mag = 0.0                 # 디버깅 확인용 최고 가속도 수치
        self.last_active_time = 0.0         # 마지막으로 흔들림이 감지된 시각

        print(f"[{self.port}] 연결 완료 (무한 반복 / 초강력 흔들기 정지 모드).")

    def loop(self):
        if self.ser.in_waiting <= 0:
            time.sleep(0.005)
            return

        while self.ser.in_waiting > 0:
            line = self.ser.readline().decode("utf-8", errors="ignore").strip()
            parts = line.split()
            if len(parts) != 3:
                continue

            try:
                x, y, z = (float(n) for n in parts)
            except ValueError:
                continue

            raw_magnitude = math.hypot(x, y, z)
            self.window.append(raw_magnitude)

        if len(self.window) < 5:
            return

        smooth_magnitude = sum(self.window) / len(self.window)

        match smooth_magnitude:
            case m if m >= self.TH_STOP:
                level = 4
            case m if m >= self.TH_STRONG:
                level = 3
            case m if m >= self.TH_MEDIUM:
                level = 2
            case m if m >= self.TH_WEAK:
                level = 1
            case _:
                level = 0

        now = time.monotonic()

        if level > 0:
            if level > self.peak_level:
                self.peak_level = level
            if smooth_magnitude > self.peak_mag:
                self.peak_mag = smooth_magnitude
            self.last_active_time = now

        elif self.peak_level > 0 and (now - self.last_active_time) >= self.SETTLE_SEC:
            target_flag = self.peak_level
            recorded_peak = self.peak_mag
            self.peak_level = 0
            self.peak_mag = 0.0

            # 4단계(초강력 흔들기): 음악 즉시 정지 및 플래그 초기화
            if target_flag == 4:
                was_playing = self.confirmed_flags is not None
                self._stop_music()
                self.confirmed_flags = None

                if was_playing:
                    print(f"[음악 정지] 초강력 흔들기 감지 (최고 강도: {recorded_peak:.1f} m/s^2)")
                else:
                    print(f"[대기 유지] 이미 정지 상태입니다 (최고 강도: {recorded_peak:.1f} m/s^2)")
                return

            # 1~3단계: 현재 재생 중인 단계와 다를 때만 새 음악 재생
            if target_flag != self.confirmed_flags:
                print(f"Confirmed flag: {target_flag} (최고 강도: {recorded_peak:.1f} m/s^2)")

                tracks = {
                    1: "beatles_yellow_submarine.mp3",
                    2: "led_zeppelin_stairway_to_heaven.mp3",
                    3: "acdc_thunderstruck.mp3",
                }
                if self._play(tracks[target_flag]):
                    self.confirmed_flags = target_flag

    def _looping_stream(self, filepath: Path):
        """MP3 파일이 끝나면 자동으로 처음부터 다시 이어 붙이는 무한 반복 제너레이터"""
        required_frames = yield b""
        raw_stream = None
        try:
            while True:
                raw_stream = miniaudio.stream_file(
                    str(filepath),
                    output_format=self.SAMPLE_FORMAT,
                    nchannels=self.CHANNELS,
                    sample_rate=self.SAMPLE_RATE,
                )
                next(raw_stream)
                while True:
                    try:
                        chunk = raw_stream.send(required_frames)
                    except StopIteration:
                        break
                    required_frames = yield chunk
        finally:
            if raw_stream is not None:
                raw_stream.close()

    def _stop_music(self):
        """순수하게 오디오 출력과 스트림 자원만 정리합니다 (플래그 상태는 변경하지 않음)."""
        if self.device is not None:
            self.device.stop()
        if self._active_stream is not None:
            self._active_stream.close()
            self._active_stream = None

    def _play(self, filename: str) -> bool:
        filepath = self.BASE_DIR / filename
        if not filepath.is_file():
            print(f"[경고] 파일을 찾을 수 없습니다: {filepath.name}")
            return False

        self._stop_music()

        stream = self._looping_stream(filepath)
        next(stream)

        self._active_stream = stream
        self.device.start(self._active_stream)
        return True

    def cleanup(self, *args):
        print("\n프로그램을 종료합니다.")
        self._stop_music()
        self.confirmed_flags = None
        if self.device is not None:
            self.device.close()
            self.device = None
        if self.ser is not None and self.ser.is_open:
            self.ser.close()
        super().cleanup()


if __name__ == "__main__":
    SimpleMp3Player()
```
