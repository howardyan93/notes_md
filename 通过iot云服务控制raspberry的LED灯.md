通过iot云服务控制raspberry的LED灯
============================================
已经被人做烂的实验，但是闲得蛋疼自己也来搞一遍。。

iot云服务里面比较有名的yeelink API操作很蛋疼。POST各种拒绝，于是我决定用wsncloud来搞。

代码如下：

    import requests
    import json
    import time
    import RPi.GPIO as GPIO
    
    led_pin = 11
    GPIO.setwarnings(False)
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(led_pin, GPIO.OUT)
    
    while(True):
    url = ‘http://api.wsncloud.com/data/v1/show’
    key = ‘your key’
    node_id = ‘your node id’
    payload = {‘ak’:key, ‘sensorId’:node_id}
    req = requests.request(‘GET’, url, params=payload)
    message = json.loads(req.text)
    if message['value'] == 1:
    #print ‘Turn on the light’
    GPIO.output(led_pin, 1)
    else:
    #print ‘Turn on the light’
    GPIO.output(led_pin, 0)
    time.sleep(2)

因为是向服务器发送GET请求，速度延迟厉害。而且云服务器也不允许频繁的请求。yeelink现在已经开始实验mqtt协议了，但是我下一步实验是采用xmpp来搞及时的控制。

