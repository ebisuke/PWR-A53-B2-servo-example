# PWR-A53-B2-servo-example
PWR-A53-B2 reverse engineered servo control example code

https://qiita.com/ebisuke/items/223309882abda44fd1c7

# Summary
I bought the XiaoR GEEK Raspberry Pi TH Robot and was ready to start using it, but the included library didn't work, so I had to find a way to operate it myself. Unfortunately, there was no information available such as source code or documentation, so I had to do some research.

# Challenges
Initially, I faced the following issues:
- The included library was for Python 2 and 32-bit, making it incompatible with the latest OS.
- The driver board that came with it is the PWR-A53-B2, but the documentation is for an older version, targeting the PWR-A53-A. According to the circuit diagram, the servo signal and Raspberry Pi are directly connected, but in reality, there is an IC in between.
- The communication with this IC seemed to be through UART according to the documentation, but it appeared to be different, more like I2C.
- Even when I tried to operate it myself, there were no specifications for communication available.

# What I Did
## i2cdetect
I ran i2cdetect somewhat randomly and got a response at 0x17.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/549340/c758b771-1f2a-1e30-4d04-db31ce2433fd.png)  
Since nothing else was connected, I assumed this was the IC on the driver board.

## Control
I tried writing to it randomly, but there was no response. However, after running i2cdump several times, the servo's torque dropped to zero, which was a mysterious behavior. After more research, I discovered:
- It seems to be controlled by reading from i2c.
- It appears to operate when a specific frame sequence is read from the memory address.

So, I concluded that reading the following frame sequence might allow some control:

| 0 | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | ---
| 0xFF | 0xFF | 0xFF | Command Type | Value | 0xFF |
- Command Type: The object to be controlled
- Value: Control value, possibly ranging from 0x00 to 0xFE?

As mentioned earlier, running i2cdump seems to reset the IC, reducing the torque to zero.

### Command Types
Please note that these are all my assumptions.
| Type Number | Value | Content |
| --- | --- | --- |
| 0x01 | Angle 00-FE | Servo1 Angle |
| 0x02 | Angle 00-FE | Servo2 Angle |
| 0x03 | Angle 00-FE | Servo3 Angle |
| 0x04 | Angle 00-FE | Servo4 Angle |
| 0x05 | Angle 00-FE | Servo5 Angle |
| 0x06 | Angle 00-FE | Servo6 Angle |
| 0x07 | Angle 00-FE | Servo7 Angle |
| 0x08 | Angle 00-FE | Servo8 Angle |

# Sample Code
This code moves each servo in the range of 0x10-0x70.

```python
import time
import smbus

if __name__ == '__main__':
    data=[
        [0xff,0xff,0xff,0x01,0x10,0xff,],
        [0xff,0xff,0xff,0x01,0x70,0xff,],
        [0xff,0xff,0xff,0x02,0x10,0xff,],
        [0xff,0xff,0xff,0x02,0x70,0xff,],
        [0xff,0xff,0xff,0x03,0x10,0xff,],
        [0xff,0xff,0xff,0x03,0x70,0xff,], 
        [0xff,0xff,0xff,0x04,0x10,0xff,],
        [0xff,0xff,0xff,0x04,0x70,0xff,],
        [0xff,0xff,0xff,0x05,0x10,0xff,],
        [0xff,0xff,0xff,0x05,0x70,0xff,],
        [0xff,0xff,0xff,0x06,0x10,0xff,],
        [0xff,0xff,0xff,0x06,0x70,0xff,],
        [0xff,0xff,0xff,0x07,0x10,0xff,],
        [0xff,0xff,0xff,0x07,0x70,0xff,],
        [0xff,0xff,0xff,0x08,0x10,0xff,],
        [0xff,0xff,0xff,0x08,0x70,0xff,],
    ]
    i2c=smbus.SMBus(1)
    addr=0x17
    for d in data:
        print(d)
        for c in d:
            # read
            _=i2c.read_byte_data(addr,c)
            time.sleep(0.05)
        time.sleep(1)

```
Video  
![img.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/549340/f538eec0-0d0d-a3a2-97f5-3810e26f307e.gif)