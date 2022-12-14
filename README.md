# PicoW - PracticaWi-Fi

**Código:**

    from machine import Pin
    import time
    import socket
    import network

    led = Pin("LED", Pin.OUT)

    ssid = 'Mora'
    password = 'MOEM001212'

    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    html = """<!DOCTYPE html>
    <html>
    <head> <title>Sistemas Programables</title> </head>
    <body> <h1>Pico W</h1>
    <p>Hola mundo</p>
    <p>Perez Ramirez Owen Osvaldo 19211707</p>
    </body>
    </html>
    """

    # Wait for connect or fail
    max_wait = 100
    while max_wait > 0:
    if wlan.status() < 0 or wlan.status() >= 3:
        break
    max_wait -= 1
    print('waiting for connection...')
    time.sleep(1)

    # Handle connection error
    if wlan.status() != 3:
        raise RuntimeError('network connection failed')
    else:
        print('connected')
        status = wlan.ifconfig()
        print('ip = ' + status[0])

    # Open socket
    addr = socket.getaddrinfo('0.0.0.0', 80)[0][-1]

    s = socket.socket()
    s.bind(addr)
    s.listen(1)

    print('listening on', addr)

    def blink_led():
        led.on()
        time.sleep(0.2)
        led.off()
        time.sleep(0.2)
        led.on()
        time.sleep(0.2)
        led.off()
        time.sleep(0.2)
        led.on()
        time.sleep(0.2)
        led.off()

    # Listen for connections
    while True:
        try:
            cl, addr = s.accept()
            blink_led()
            print('client connected from', addr)
            cl_file = cl.makefile('rwb', 0)
            while True:
                line = cl_file.readline()
                if not line or line == b'\r\n':
                    break
            response = html
            cl.send('HTTP/1.0 200 OK\r\nContent-type: text/html\r\n\r\n')
            cl.send(response)
            cl.close()

        except OSError as e:
            cl.close()
            print('connection closed')

**Imágenes:**

https://user-images.githubusercontent.com/105949773/202311601-825057d4-d8d9-4321-8f70-0063837fdacd.mp4
