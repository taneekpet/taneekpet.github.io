---
title: Mini-blog on data breach
tags:
  - Data breach
  - Security
date: 2019-01-19 11:00:00 +0700
---

จาก[ข่าวข้อมูลหลุด][blognone] ครั้งล่าสุดที่มี email และรหัสผ่านชุดใหญ่หลุดออกมา

หลายๆคนคงได้รู้จักเว็บ [Have I been pwned][hibp] 
ที่เอาไว้เช็คว่าข้อมูลที่เคยหลุดๆออกมานั้นกระทบเราบ้างหรือเปล่า

และก็คงได้ลองเอา email ตัวเองไปเช็คแล้วอาจจะเกิดสงสัยว่า
เว็บยักษ์ใหญ่อย่าง hotmail หรือ gmail ทำไมโดนแฮกได้ง่ายจัง

คำตอบคือไม่ใช่ครับ จริงๆคือหลายๆบริการอนุญาตให้เราใช้ email แทน username ได้
และถ้าเว็บพวกนั้นโดนแฮก email ของเราจะหลุดสู่สาธารณะได้ *แต่ตัว email นั้นไม่ได้โดน*

![pwned]({{ site.baseurl }}/images/pwned.jpg)
*ดูด้านล่างจะเห็นว่าโดนจาก dropbox ไม่ใช่ hotmail*

เช่นจากรูปนี้ อ่านข้างล่างจะเห็นว่าข้อมูลหลุดมาจาก Dropbox
*ซึ่งสิ่งที่ควรทำไม่ใช่เปลี่ยนรหัส email แต่คือการเปลี่ยนรหัส dropbox*

เพราะฉะนั้นใครที่เอา email ตัวเองไปเช็คแล้วได้รับผลกระทบ อย่าเพิ่งรีบเปลี่ยนรหัส email แล้วสบายใจ
เลื่อนลงมาอ่านข้างล่างนิดหนึ่งว่าเราได้รับผลกระทบจากบริการไหนแน่ จะได้แก้ไขให้ถูกจุดครับ

**เพิ่มเติม**

แต่ถ้าใครที่ใช้รหัสผ่านซ้ำกันหลายบริการอันนี้แนะนำให้เปลี่ยนทุกอันนะครับ

เช่น ถ้าคุณใช้รหัส facebook, email, dropbox เหมือนกัน แล้วเห็นว่าหลุดที่ dropbox
อันนี้ต้องเปลี่ยนทั้งสามอัน

Fun Fact
====

Pwned เป็นแสลงซึ่งแผลงมาจากสำนวน Owned ที่แปลไทยประมาณว่า เสร็จฉันล่ะ
และเคยมีคนพิมพ์ผิดเพราะ o กับ p อยู่ข้างๆกันบนคีย์บอร์ด
แล้วมีคนเอามาล้อมากๆเข้าจนกระจายเป็นวงกว้าง

[blognone]: //www.blognone.com/node/107631
[hibp]: //haveibeenpwned.com/