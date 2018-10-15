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
โพสต์นี้จะอธิบายเชิงเทคนิคว่า NDID ทำงานอย่างไร

[ข้อมูลโดยละเอียดหาได้ที่เว็บหลักของข้อมูลทางเทคนิค][home]

## ภาพรวมของระบบ NDID

![ndid-architecture-zoom-out]({{ site.baseurl }}/images/ndid-architecture-zoom-out.jpg)

รูปทางด้านซ้ายคือการเชื่อมต่อของแต่ละองค์กร ซึ่งจะเห็นว่าส่วนที่ติดต่อกันคือส่วนของ NDID Node
ซึ่ง NDID Node คือซอฟต์แวร์ส่วนที่บริษัท National Digital ID จำกัด จัดทำขึ้น
และอนุญาตให้สมาชิกนำไปใช้ได้ (มีแบบ docker ให้ด้วย)

การพัฒนาระบบนี้นั้นจัดทำเป็นแบบ opensource [ดูซอร์สโค้ดทั้งหมดที่นี่][repo_all]

ซึ่งเนื่องจากระบบนี้มีภาระผูกพันทางกฎหมาย การจะเข้าเป็นหนึ่งในสมาชิกจึงต้องมีการสมัครและเซ็นต์สัญญา
รวมถึงต้องยอมรับการ audit ต่างๆตามที่บริษัท NDID กำหนดด้วย

และด้านขวาคือส่วนประกอบของแต่ละโมดูลใน NDID Node

## Architecture ของระบบ

![ndid-architecture-zoom-in]({{ site.baseurl }}/images/ndid-architecture-zoom-in.jpg)

Repositories หลักของระบบนี้ที่แนะนำให้สมาชิกนำไปรันประกอบด้วย

- [API][repo_api] ซึ่งเป็นส่วนที่ติดต่อกับแอพลิเคชั่นของสมาชิกผ่านทาง REST API
  - เขียนด้วย Nodejs
- [Smart-contract][repo_smart_contract] เป็นส่วนที่ sync ข้อมูลสาธารณะกันระหว่างสมาชิกทั้งหมดด้วย blockchain
  - เขียนด้วย golang
  - **ไม่มีการเก็บ sensitive data บน blockchain**

ซึ่งระหว่าง API และ Smart-contract จะเชื่อมกันด้วย JSON-RPC over HTTP และ Websocket

นอกเหนือจากสอง repositories หลักด้านบนแล้ว ยังประกอบด้วย
- [test][repo_test] ที่เอาไว้รัน end-to-end ก่อนที่จะ release แต่ละเวอร์ชั่น
- [examples][repo_examples] ซึ่งจำลองการทำงานเบื้องต้นของแอพลิเคชั่นของสมาชิก
  - ใช้เพื่อง่ายในการสื่อสารให้เห็นภาพในช่วงแรกๆของการพัฒนาระบบ

ซึ่งโพสต์นี้จะอธิบายเฉพาะส่วนของสอง repositories หลักเท่านั้น

## API Repository

เป็นส่วนที่ติดต่อกับแอพลิเคชั่นของสมาชิกด้วย REST API พัฒนาด้วย Nodejs
- ทำหน้าที่เช็คความถูกต้องของข้อมูลที่สมาชิกส่งลงมาก่อนส่งต่อลงไปยัง blockchain
- ทำหน้าที่จัดการการคำนวณทาง crytography ต่างๆ
- ทำหน้าที่จัดการ Distributed Message สำหรับส่ง sensitive data ระหว่างสมาชิก
- ทำหน้าที่จัดการ cache เพื่อให้ process สามารถทำงานต่อได้หลังจาก restart

Process ในส่วนนี้จะรันอยู่บนเครื่องของสมาชิก
และสมาชิกจะต้องตั้งค่าไฟวอลล์ให้เครื่องรับการเชื่อมต่อจากเครื่องใน network ที่ไว้ใจได้เท่านั้น

ส่วนที่ API Process เชื่อมต่อกับ Internet จะมีแค่ส่วนของ Distributed Message (ZMQ)

### API และ Node Logic

- validate
- prepare data to blockchain, encrypt
- call blockchain
- encrypt, manange mq
- etc.

### Redis DB

- short term cache
  - what in this
  - to store mq/blockchain to wait for another
- long term cache
  - what in this

### Distributed Message (ZMQ)

- how encrypt/decrypt
- exactly once

## Smart-contract repository

เป็นส่วนที่จัดการ blockchain เพื่อให้สมาชิกทั้งวงมีข้อมูลที่ตรงกัน

ข้อมูลในส่วนนี้จะเป็นข้อมูลเปิดเผยและ**ไม่มีการเก็บ plain sensitive data ใดๆ ลง blockchain ทั้งสิ้น**

การจะอ่านข้อมูลจาก blockchain นี้ เพียงแค่รู้ genesis file และ chain id ก็สามารถอ่านข้อมูลได้
แต่การจะตั้ง blockchain node ที่สามารถสร้าง transaction ใดๆลง blockchain 
จะต้องได้รับการ approve จากบริษัท NDID ก่อนเท่านั้น

(ยังไม่มีการสรุปว่าบริษัทฯจะเปิดเผย genesis file และ chain id เป็นสาธารณะหรือไม่)

โดย blockchain ที่เลือกนำมาใช้ในระบบนี้คือ [tendermint][tendermint]

### Tendermint

- voting power
- check with ABCI
- node id
- hash in consistency cause disconect

### ABCI

- check tx, deliver tx

### App State DB

- store data of ABCI

## ต่อเชื่อมกันยังไง

- callback
- อธิบาย flow

[home]: //ndidplatform.github.io/
[non_technical]: /2018/10/14/introduction-ndid.html

[repo_all]: //github.com/ndidplatform
[repo_api]: //github.com/ndidplatform/api
[repo_smart_contract]: //github.com/ndidplatform/smart-contract
[repo_test]: //github.com/ndidplatform/test
[repo_examples]: //github.com/ndidplatform/examples

[tendermint]: //www.tendermint.com/