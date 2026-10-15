
import time
import ctypes
from ctypes import wintypes
import random


class POINT(ctypes.Structure):
    _fields_ = [
        ("x", wintypes.LONG),
        ("y", wintypes.LONG),
    ]


user32 = ctypes.windll.user32
SCREEN_WIDTH = user32.GetSystemMetrics(0)
SCREEN_HEIGHT = user32.GetSystemMetrics(1)

# 鼠标微动范围
MIN_STEP = 8
MAX_STEP = 45
# 主循环间隔（随机）
MIN_INTERVAL = 3
MAX_INTERVAL = 9
# 按键触发概率（百分比）
KEY_TRIGGER_RATE = 32
# 鼠标点击触发概率
CLICK_TRIGGER_RATE = 18


def get_mouse_position():
    point = POINT()
    user32.GetCursorPos(ctypes.byref(point))
    return point.x, point.y


def set_mouse_position(x, y):
    user32.SetCursorPos(int(x), int(y))


def simulate_key_press():
    """随机模拟Shift轻按，模拟无意识按键"""
    VK_SHIFT = 0x10
    user32.keybd_event(VK_SHIFT, 0, 0, 0)
    time.sleep(random.uniform(0.04, 0.18))
    user32.keybd_event(VK_SHIFT, 0, 0x0002, 0)


def simulate_mouse_click():
    """模拟左键单击，模拟真人点击空白处"""
    # 左键按下
    user32.mouse_event(2, 0, 0, 0, 0)
    time.sleep(random.uniform(0.06, 0.2))
    # 左键松开
    user32.mouse_event(4, 0, 0, 0, 0)


def curve_move_to(start_x, start_y, target_x, target_y, steps=12):
    """
    曲线平滑移动：模拟人手缓慢移动，而不是瞬间瞬移
    """
    for i in range(1, steps + 1):
        ratio = i / steps
        # 加入正弦曲线扰动，模拟手腕自然抖动
        curve_offset = random.uniform(-2.2, 2.2) * (1 - ratio)
        cur_x = start_x + (target_x - start_x) * ratio + curve_offset
        cur_y = start_y + (target_y - start_y) * ratio + curve_offset
        set_mouse_position(cur_x, cur_y)
        time.sleep(random.uniform(0.015, 0.035))


def calculate_safe_position(x, y):
    step = random.randint(MIN_STEP, MAX_STEP)
    direction = random.choice(["right", "left", "up", "down"])

    if direction == "right" and x + step < SCREEN_WIDTH:
        return x + step, y
    elif direction == "left" and x - step >= 0:
        return x - step, y
    elif direction == "up" and y - step >= 0:
        return x, y - step
    elif direction == "down" and y + step < SCREEN_HEIGHT:
        return x, y + step
    else:
        return calculate_safe_position(x, y)


def main():
    print("Human-like mouse mover started. Ctrl+C to stop.")
    try:
        while True:
            orig_x, orig_y = get_mouse_position()
            dest_x, dest_y = calculate_safe_position(orig_x, orig_y)

            # 平滑曲线移动过去
            curve_move_to(orig_x, orig_y, dest_x, dest_y)
            time.sleep(random.uniform(0.08, 0.22))

            # 随机触发单击动作（18%概率）
            if random.randint(1, 100) <= CLICK_TRIGGER_RATE:
                time.sleep(random.uniform(0.2, 0.8))
                simulate_mouse_click()

            # 平滑移回原位
            curve_move_to(dest_x, dest_y, orig_x, orig_y)

            # 随机概率触发按键，非固定出现
            if random.randint(1, 100) <= KEY_TRIGGER_RATE:
                time.sleep(random.uniform(0.4, 1.4))
                simulate_key_press()

            # 下一轮等待随机时长
            wait = random.randint(MIN_INTERVAL, MAX_INTERVAL)
            time.sleep(wait)
    except KeyboardInterrupt:
        print("\nStopped.")


if __name__ == "__main__":
    main()
