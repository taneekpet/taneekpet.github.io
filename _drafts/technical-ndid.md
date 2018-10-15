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
*ภาพจาก NDID installation guideline*

รูปทางด้านซ้ายคือการเชื่อมต่อของแต่ละองค์กร ซึ่งจะเห็นว่าส่วนที่ติดต่อกันคือส่วนของ NDID Node
ซึ่ง NDID Node คือซอฟต์แวร์ส่วนที่บริษัท National Digital ID จำกัด จัดทำขึ้น
และอนุญาตให้สมาชิกนำไปใช้ได้ (มีแบบ docker ให้ด้วย)

การพัฒนาระบบนี้นั้นจัดทำเป็นแบบ opensource [ดูซอร์สโค้ดทั้งหมดที่นี่][repo_all]

ซึ่งเนื่องจากระบบนี้มีภาระผูกพันทางกฎหมาย การจะเข้าเป็นหนึ่งในสมาชิกจึงต้องมีการสมัครและเซ็นต์สัญญา
รวมถึงต้องยอมรับการ audit ต่างๆตามที่บริษัท NDID กำหนดด้วย

และด้านขวาคือส่วนประกอบของแต่ละโมดูลใน NDID Node

## Architecture ของระบบ

![ndid-architecture-zoom-in]({{ site.baseurl }}/images/ndid-architecture-zoom-in.jpg)
*ภาพจาก NDID installation guideline*

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

## คู่ key ที่สมาชิกต้องเก็บรักษา

เมื่อสมาชิกทำการเซ็นสัญญากับบริษัท NDID แล้ว
สมาชิกจะทำการสร้างคู่ private/public key จำนวน 3 คู่

1. คู่ key ที่ใช้สำหรับประมวลผลภายในของ blockchain และใช้แทนสิทธ์ในการ vote ว่า block 
หนึ่งๆเป็น block ที่ถูกต้องใน blockchain หรือไม่ (tendermint key)

2. คู่ key ที่ใช้เพื่อยืนยันว่า transaction ในระบบมาจากสมาชิกนี้จริงๆ 
และใช้เพื่อ encrypt/decrypt ข้อมูลที่ส่งผ่านทาง message queue (node key)

3. คู่ key ที่ไม่ใช้เพื่อการอื่นนอกจากใช้เพื่อเปลี่ยน node key ในข้อ 2 (master key)
หรือเพื่อเปลี่ยน master key เองเท่านั้น

สมาชิกจะส่ง public key ของทั้งสามคู่ให้บริษัทเพื่อนำเข้าระบบและประกาศให้สมาชิกอื่นๆทราบ
ส่วน private key ทั้งสามคู่สมาชิกต้องเก็บรักษาเป็นความลับ

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

เป็น HTTP server ที่ให้แอพลิเคชั่นของสมาชิกติดต่อมาเมื่อต้องการทำธุรกรรมผ่าน NDID
และสมาชิกจะต้องการตั้งค่า URL เพื่อให้ process ติดต่อกลับไปในกรณีต่างๆ เช่น

- มีการเปลี่ยนแปลงเกิดขึ้นกับธุรกรรมที่สมาชิกเกี่ยวข้อง
- ส่งข้อมูลที่ต้องการให้สมาชิกทำการสร้าง digital signature ด้วย private key ของสมาชิก
- ส่งข้อมูลที่ได้รับจากสมาชิกอื่นผ่านทาง message queue เพื่อให้สมาชิก decrypt ด้วย private key ของตน

**_Application -> API process_**

เมื่อได้รับ request จากสมาชิก
Node logic จะทำการตรวจสอบรูปแบบข้อมูลว่าตรงตาม API standard ที่ประกาศไว้หรือไม่

หากข้อมูลนั้นถูกต้อง Node Logic จะทำการจัดรูปแบบข้อมูลให้ถูกต้องตามรูปแบบที่จะจัดเก็บลง blockchain และส่งคืนให้สมาชิกทำการสร้าง digital signature เพื่อยืนยัน

เมื่อได้ข้อมูลที่ยืนยันตัวสมาชิกได้จาก digital signature แล้ว Node logic จะทำการส่งข้อมูลไปยัง blockchain เพื่อบันทึกไว้เป็นหลักฐานว่ามีธุรกรรมเกิดขึ้น

หลังจากนั้นหากต้องมีการติดต่อส่ง sensitive data ไปยังสมาชิกรายอื่น Node Logic จะนำ public key ของสมาชิกปลายทางมา encrypt ข้อมูลแล้วส่งต่อทาง message queue

**_API process -> Application_**

เมื่อ Node logic ได้รับข้อมูลผ่านทาง message queue แล้ว
Node logic จะส่งต่อให้สมาชิกเพื่อทำการ decrypt และอ่านข้อมูลหลังจาก decrypt เก็บพักไว้ใน short term cache

หาก Node logic มองเห็นเปลี่ยนแปลงจาก blockchain ถึงธุรกรรมที่เกี่ยวข้องกับสมาชิก
และมีข้อมูลที่เกี่ยวข้องหลังจาก decrypt อยู่ใน short term cache แล้ว
Node logic จะทำการตรวจสอบความถูกต้องของข้อมูล
รวมถึงตรวจสอบว่าข้อมูลที่ได้มาจากสมาชิกรายได้ ผ่านทาง digital signature

หากข้อมูลถูกต้องทั้งหมด จะทำการส่งต่อให้สมาชิกในรูปแบบที่ได้ประกาศเป็น standard API ไว้

### Redis DB

ในการทำงานของ Node logic จำเป็นต้องใช้ข้อมูลจากทั้งสองช่องทางนั่นคือ message queue และ blockchain
แต่ข้อมูลจากทั้งสองช่องทางอาจจะมาถึงไม่พร้อมกัน จึงต้องมี cache เอาไว้พักข้อมูลที่มาถึงก่อนเอาไว้

- short term cache
  - ใช้เพื่อพักข้อมูลที่เกี่ยวข้องจากช่องทางใดช่องทางหนึ่ง
  - ใช้เพื่อเก็บสถานะของ process ว่าประมวลผลถึงขั้นใด 
  สำหรับกรณีที่ process เกิดการ crash แล้ว restart จะได้ยังสามารถทำงานต่อจากจุดเดิมได้
- long term cache
  - ใช้เพื่อพักข้อมูลที่ได้รับและส่งออกผ่านทาง message queue เพื่อเก็บไว้เป็นหลักฐานตามกฎหมาย
  - สมาชิกจะต้องดึงข้อมูลออกและนำไปเก็บในที่ปลอดภัยแล้วจึงสั่งลบข้อมูลในส่วนนี้ออกจาก cache
  - การดึงข้อมูลและลบ สามารถสั่งผ่าน REST API ได้

### Distributed Message (ZMQ)

- ใช้เพื่อส่ง sensitive data ระหว่างสมาชิก
- เป็นการส่งแบบ P2P
- มีการสร้าง digital signature ของผู้ส่ง และข้อมูลทั้งหมดถูก encrypt ด้วย public key ของผู้รับ
- ข้อมูลใดๆที่ส่งผ่าน message queue จะมีการบันทึกค่า hash ลง blockchain ไว้เป็นหลักฐาน

ย้ำอีกครั้งว่าข้อมูลที่ถูกบันทึกลง blockchain คือ hash ของ sensitive data ไม่ใช่ sensitive data โดยตรง

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