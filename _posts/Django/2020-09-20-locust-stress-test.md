---
layout: post
title: "[Django] 장고 서버 locust로 부하 테스트"
author: qwlake
categories: Django
tags: Django locust stress-test
---

![image](https://user-images.githubusercontent.com/41278416/106115342-0876f580-6194-11eb-98d5-137119a96eba.png)

### 사진 설명

**차트1 (Total Requests per Second)**
green line: RPS,
red line: Failures/s

**차트2 (Response Times (ms))**
green line: Medium Response Time,
yellow line: 95% percentile

**차트3(Number of Users)**
green line: Users

### Task 생명 주기

1. Locust setup
2. TaskSet setup
3. TaskSet on_start
4. TaskSet tasks…
5. TaskSet on_stop
6. TaskSet teardown
7. Locust teardown

### Task 코드

```python
import string
import random

from locust import HttpUser, task, between

def id_generator(size=6, chars=string.ascii_uppercase + string.digits):
    return ''.join(random.choice(chars) for _ in range(size))

class QuickstartUser(HttpUser):
    wait_time = between(1, 5)
    device_id_list = []

    @task(3)
    def get_search(self):
        self.client.get("/notice/all?target=cse+main&q=장학")

    @task(3)
    def get_notices(self):
        for i in range(1,11):
            self.client.get(f"/notice/all?target=cse+main&page={i}")

    @task(1)
    def put_subs(self):
        with self.client.put(
            "/accounts/device", 
            json={
                "id":random.choice(self.device_id_list), 
                "subscriptions":"cse+main"}) as response:
            if not response.ok:
                print(response)

    @task(1)
    def put_keywords(self):
        with self.client.put(
            "/accounts/device", 
            json={
                "id":random.choice(self.device_id_list), 
                "keywords":"장학+등록"}) as response:
            if not response.ok:
                print(response)

    @task(1)
    def put_alarm(self):
        with self.client.put(
            "/accounts/device", 
            json={
                "id":random.choice(self.device_id_list), 
                "alarm_switch":random.choice([True,False])}) as response:
            if not response.ok:
                print(response)

    @task(1)
    def get_notice_list(self):
        for i in range(5):
            self.client.get("/notice/list")

    def on_start(self):
        device_id = id_generator(30)
        self.device_id_list.append(device_id)
        self.client.post("/accounts/device", json={"id":device_id})
```

### (참고용) Web server, middle ware 유무에 따른 변화

![image](https://user-images.githubusercontent.com/41278416/106115367-0e6cd680-6194-11eb-8ea3-8b08a4b03bb8.png)

with nothing, with gunicorn, with guniforn+nginx

### References

[https://docs.locust.io/en/stable/api.html#response-class](https://docs.locust.io/en/stable/api.html#response-class)

[https://bcho.tistory.com/1369](https://bcho.tistory.com/1369)