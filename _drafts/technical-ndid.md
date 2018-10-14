---
title: เรื่องทางเทคนิคของ NDID
tags:
  - NDID
  - Government
  - Identity
  - Blockchain
  - Tendermint
  - Nodejs
  - Golang
  - Redis
  - ZMQ
  - Opensource
date: 2018-10-03 19:45:00 +0700
---

จาก[โพสต์ที่แล้ว][non_technical]ที่อธิบายคร่าวๆว่า NDID คืออะไร

โพสต์จะอธิบายเชิงเทคนิคว่า NDID ทำงานอย่างไร คนที่อยากต่อเชื่อมกับ NDID จะทำอย่างไร
[เว็บหลักของข้อมูลทางเทคนิค][home]

ภาพรวมของระบบ NDID
----

![ndid-architecture-zoom-out]({{ site.baseurl }}/images/ndid-architecture-zoom-out.jpg)

รูปทางด้านซ้ายคือการเชื่อมต่อของแต่ละองค์กร ซึ่งจะเห็นว่าส่วนที่ติดต่อกันคือส่วนของ NDID Node
ซึ่ง NDID Node คือซอฟต์แวร์ส่วนที่บริษัท National Digital ID จำกัด จัดทำขึ้น
และให้สมาชิกนำไปใช้ (มีแบบ docker ให้ด้วย)

ซึ่งเนื่องจากระบบนี้มีภาระผูกพันทางกฎหมาย การจะเข้าเป็นหนึ่งในสมาชิกจึงต้องมีการสมัครและเซ็นสัญญา
รวมถึงต้องยอมรับการ audit ต่างๆจากบริษัท NDID ด้วย

และด้านขวาคือส่วนประกอบของแต่ละโมดูลใน NDID Node

Architecture ของระบบ
----

![ndid-architecture-zoom-in]({{ site.baseurl }}/images/ndid-architecture-zoom-in.jpg)

Repository หลักของระบบนี้ประกอบด้วย

- [API][repo_api] ซึ่งเป็นส่วนที่ติดต่อกับแอพลิเคชั่นของสมาชิกผ่านทาง REST API
  - เขียนด้วย Nodejs
- [Smart-contract][repo_smart_contract] เป็นส่วนที่ sync ข้อมูลกันระหว่างสมาชิกทั้งหมดด้วย blockchain
  - เขียนด้วย golang
  - **ไม่มีการเก็บ sensitive data บน blockchain**

ซึ่งระหว่าง API และ Smart-contract จะเชื่อมกันด้วย JSON-RPC over HTTP และ Websocket

หน้าที่ของแต่ละโมดูล
----

API
---


เทคโนโลยีของแต่ละโมดูล
----

ต่อเชื่อมกันยังไง
----



[home]: //ndidplatform.github.io/
[non_technical]: /2018/10/14/introduction-ndid.html
[repo_api]: //github.com/ndidplatform/api
[repo_smart_contract]: //github.com/ndidplatform/smart-contract