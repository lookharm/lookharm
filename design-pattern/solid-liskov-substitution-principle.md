---
description: '#clean-code #solid #design-pattern'
---

# SOLID: Liskov Substitution Principle

**2022-02-15**

LSP ในความเข้าใจของข้าพเจ้า

เนื่องจากเป็นข้อที่อ่านนิยามแล้วงงสุดๆ เลยพยายามที่จะหาข้อมูลจากหลายๆ ที่มาลองเปรียบเทียบ เพื่อให้เห็นว่าแต่ละคนที่เขียน LSP ออกมาจะเข้าใจมันไปทางไหนบ้าง

เมื่อได้ลองอ่านไปหลายๆ ที่พบว่าทั้งหมดมีมีลักษณะร่วมกันบางอย่างคือ

1. การให้ความหมาย
2. ตัวอย่างที่ผิด
3. วิธีแก้
4. ถ้าไม่ทำตามจะเจออะไร
5. ถ้าทำตามจะได้อะไร

> บางข้อความอาจคัดลอกมาทั้งดุ้น บางข้อความเป็นความเห็นส่วนตัว

### Design Principles and Design Patterns - Robert C. Martin

[Design Principles and Design Patterns - Robert C. Martin](http://staff.cs.utu.fi/\~jounsmed/doos\_06/material/DesignPrinciplesAndPatterns.pdf)

เริ่มกันที่ต้นฉบับเลยก็แล้วกัน Rober C. Martin อาจไม่ใช่คนแรกที่พูดถึง Liskov Substitution แต่เป็นคนแรกๆ ที่พูดเรื่อง SOLID

#### การให้ความหมาย

* Subclasses should be substitutable for their base classes.
* Coined by Barbar Liskov. Derives concept from Design by Contract of Bertrand Meye.
* Restating the LSP, we can say that, in order to be substitutable, the contract of the base class must be honored by the derived class. (ควรซื่อสัตย์กับ superclass ซึ่งเสมือน contract).
* Precondition: What must be true before the method is called.
* Postcondition: What the method guarantees will be true once it has completed.
* "expect no more and provide no less"
* Violations of LSP are latent violations of OCP.

#### ตัวอย่างที่ผิด

* Circle/Ellipse dilemma
* Circle คือ subset ของ Ellipse ที่มี foci (จุด focus point) เท่ากัน
* หากเอา Circle ไป extends Ellipse (Circle เป็น subclass ของ Ellipse) Circle ต้องถูกเรียกใช้โดย client ได้โดยไม่ส่งผลผิดพลาด
* กำหนด Ellipse: SetFoci(a: Point, b: Point)
* การ implement SetFoci() ของ Circle จะไป set ให้ a = a, b = b หาก client นำไปเรียกใช้ก็จะเกิดความผิดพลาด
* เพราะว่า Postcondition ของ SetFoci() เราควร getA() ต้องได้ a ที่ set ไป และ getB() ต้องได้ b ที่ set ไป
* eg. หาเราส่ง SetFoci(1, 9) จะทำให้ a=1 และ b=1 แทนที่จะเป็น 9

#### วิธีแก้

* Martin กล่าวว่า LSP ยากที่จะตรวจพบ
* ต้อง rebuild design
* หรือใส่ if/else statement เข้าไปใน client ที่เจอปัญหา
* การใส่ if/else statement เข้าไปทำให้ทุกครั้งทีมีการเพิ่ม Ellipse ต้องมาแก้ code ที่ฝั่ง client ด้วยนั่นคือเพิ่ม if/else เข้าไปจัดการกับ Ellipse ที่เพิ่มเข้ามา ซึ่งก็จะ break OCP

> \[opinion] ถึงตรงนี้เข้าใจว่า Design by Contract ก็สำคัญเพราะเราควรบอกชัดเจนไปเลยว่า สิ่งนี้ทำอะไรได้ ทำอะไรไม่ได้ และ follow ตามนั้น กล่าวคือ design ให้ดีตั้งแต่แรกโอกาสเกิด LSP จะน้อย

***

### Digital Ocean

[Digital Ocean](https://www.digitalocean.com/community/conceptual\_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design#interface-segregation-principle)&#x20;

ถ้าลองค้นหาคำว่า SOLID principle ในกูเกิ้ลก็จะเจอเว็บนี้เป็นเว็บแรกๆ

#### การให้ความหมาย

* Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.
* This means that every subclass or derived class should be substitutable for their base or parent class.

#### ตัวอย่างที่ผิด

* แสดงให้เห็นถึงการ implement Interface ไม่เป็นไปเป็นไปในแนวทางเดียวกัน
* supertype return float/double
* subtype return array
* จึงผิดกด LSP
* sum() ต้อง return float/double เหมือนกันทั้งสอง subtype และ supertype
* > opiniion: ในตัวอย่างเป็น PHP ซึ่ง return เป็น dynamic ได้แต่หากเป็นภาษาที่เป็น static เช่น Golang ก็จะไม่เจอปัญหาการ return แล้วทำให้ break LSP

#### วิธีแก้

* ปรับให้ sum() ของ subtype return float/double เหมือนกับ supertype
* ปรับ return type ให้เหมือนกัน

***

### Baeldung (1)

[Baeldung (1)](https://www.baeldung.com/solid-principles)

#### การให้ความหมาย

* It is arguably the most complex of the five principles.
* If class A is a subtype of class B, we should be able to replace B with A without disrupting the behavior of our program.

#### ตัวอย่างที่ผิด

* มี 2 คลาส ElectricCar และ Car
* โดย ElectricCar extend Car
* มี method turnOnEngine() ที่ ElectricCar ไม่ได้ใช้จึง implement โดยการ throw error ออกมา จึงผิดกด LSP เพราะไม่สามารถนำ ElectricCar ไปใช้แทน Car ได้
* > \[opinion] ตรงนี้หากเป็น Golang ก็ยังคงมีปัญหานี้อยู่เช่น panic แต่ถ้า return error ออกไปเลยก็จะไม่มี

#### วิธีแก้

* ไม่มี แต่เสนอว่า การผิดกฎ LSP จะแก้ได้ยาก เพราะจะเกี่ยวกับการ design supertype ให้ดีตั้งแต่แรก

> \[opinion] ถึงตรงนี้เข้าใจว่า ตัวที่ implement supertype ต้อง implement ด้วยการทำงานลักษณะที่ถูกต้องจริงๆ ห้าม implement ให้ผิดไปจากสิ่งที่ได้ตกลงกันไว้ หรือจริงๆ แล้วการออกแบบ interface ต้องดีตั้งแต่แรก ของที่จะนำมาเป็น supertype ต้องกำหนด method ที่มีพฤติกรรมร่วมกันจริงๆ ไม่งั้นจำเป็นที่จะต้อง implement medhod ที่ไม่ได้ใช้ซึ่งผิด ISP และยังใช้งานผิดพลาด พอเข้าใจแล้วว่าใน Dave กล่าวว่าให้ Golang's interface มี 1 method คือจะทำให้เข้ากับ LSP ได้ง่ายมาก เพราะตัวที่ implement จะทำได้ง่ายกว่า คือสนใจแค่ 1 method เพราะจะ substitue ได้ง่ายกว่า

***

### Baeldung (2)

[Baeldung (2)](https://www.baeldung.com/cs/liskov-substitution-principle)

#### การให้ความหมาย

* "subclasses should be substitutable for their base classes", meaning that code expecting a certain class to be used should work if passed any of this class’ subclasses.

#### ตัวอย่างที่ผิด

* อธิบายด้วยภาพ
* supertype: Vehicle, subtype: Car, Truck
* มี Garage เรียกใช้ repair() สามารถใช้กับ Vehicle ได้ทุกตัว (Car, Truck) ไม่มีปัญหาไม่ break LSP
* แต่ CarDriver เรียกใช้ drive() ใช้ได้แค่กับ Car แต่ใช้กับ Truck ไม่ได้ (เป็นการยกตัวอย่างในเชิงสมมติ) จึง break LSP

#### วิธีแก้

* แยก drive() ออกไป เป็นของ Car แทน
* ปรับ design

#### ถ้าไม่ทำตามจะเจออะไร

* Misleading the code: เข้าใจการทำงานของ Code ผิด "we should expect some behavior to work, but it doesn’t"
* Less Readable Code: ต้องคอยจัดการว่า object ไหนทำอะไรได้ อันไหนทำอะไรไม่ได้ มี branches เยอะสำหรับจัดการ subclasses ต่างๆ
* Error-Prone Code: อาจจะจับ error ไม่เจอตอน test แต่ไม่เจอตอน production

***

### Javatechonline

[Javatechonline](https://javatechonline.com/solid-principles-the-liskov-substitution-principle)

#### การให้ความหมาย

* if class A is a subtype of class B, then we should be able to replace objects of B with objects of A (i.e., objects of type A may substitute objects of type B) without changing the behavior (correctness, functionality, etc.) of our program.
* Derived types must be completely substitutable for their base types.
* The examples of behaviors that break LSP:
  * a derived class throwing an exception that the superclass does not throw.
  * a derived class has some unexpected side effects.

#### ตัวอย่างที่ผิด

* BookDelivery มี medhod getDeliveryLocations()
* PosterMapDelivery และ AudioBookDelivery extends BookDelivery
* แต่ AudioBookDelivery implement โดยไม่ได้ใส่ code อะไรไว้เลยจึงทำให้การทำงานเกิดความผิดพลาดได้ หากมีการเอา AudioBookDelivery (subtype) ไปใช้แทน BookDelivery (supertype) ตรงนี้จึง break LSP

#### วิธีแก้

* design supertype ใหม่โดยแยกเป็น
* OfflineDelivery: getDeliveryLocations()
* OnlineDelivery: getSoftwareOptions()
* PosterMapDelivery extends OfflineDelivery
* AudioBookDelivery extends OnlineDelivery

#### ถ้าไม่ทำตามจะเจออะไร

* Undefined Behavior, good at the test but bad at production.

#### ถ้าทำตามจะได้อะไร

* Easรer Maintenance:
  * > \[opinion] ไม่ต้องคอยจำว่า subtype ตัวไหนทำอะไรได้ ทำอะไรไม่ได้เพราะ likely to cause bugs.
* Code Reusability:
  * > \[opinion] ฝั่งเรียกใช้งานที่ประกาศรับ supertype สามารถเอา subtype ไปใช้ได้เลย โดยไม่ต้องกลัวความผิดพลาด
* Reduced Coupling:
  * > \[opinion] ฝั่งที่เรียกใช้งานไม่จำเป็นต้อง specific ว่าเป็น type นั้นๆ แต่ประกาศ supertype ที่ต้องการแล้วรับ subtype ที่มีความสามารถตรงกันเข้าไป

***

### ITNEXT

[ITNEXT](https://itnext.io/solid-principles-explanation-and-examples-715b975dcad4)

#### การให้ความหมาย

* This one is probably the hardest one to wrap your head around when being introduced for the first time.
* Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

#### ตัวอย่างที่ผิด

* มี supertype คือ Post: CreatePost() และ subtype คือ TagPost และ MantionPost
* แต่ MantinPost ไม่ได้ implement CreatePost() ตั้งชื่อเป็น CreateMentionPost แทนที่จะเป็น CreatePost() ตั้งชื่อผิดหรือลืม overide
* เมื่อ MantinPost นำไปใช้แทน supertype จึงทำงานผิดเพราะมันจะไป call CreatePost() ของ Post แทน
* > \[opinion] เป็นปัญหาจากการ implement ผิด คือไม่ได้ overide method CreatePost() ตาม supertype ปัญหานี้จะเกิดเฉพาะบางภาษาเท่านั้น เช่น Java, C# ที่มีการ extends แบบ explicit แต่อย่าง Golang ที่เป็น implicit คือการจะเป็น subtype ได้ต้องมี method signature ที่ตรงกับ supertype ทั้งหมดอยู่แล้ว กล่าวคือ ต้อง implement ทั้งหมดตั้งแต่แรกอยู่แล้ว จึงไม่มีปัญหาการที่ไม่ได้ overide แต่ปัญหาการ implement ให้ผิดจากวัตถุประสงค์ของ supertype ยังคงมีอยู่

#### วิธีแก้

* แก้โดย override method CreatePost()
* ไม่ได้แก้ design

***

### Dave Cheney

[Dave Cheney](https://dave.cheney.net/2016/08/20/solid-go-design)

จะเน้นโยง Golang เข้ากับ SOLID

#### การให้ความหมาย

* Two types are substitutable if they exhibit behavior such that the caller is unable to tell the difference.
* Well-designed interfaces are more likely to be small interfaces; the prevailing idiom is an interface that contains only a single method.
* พูดถึง interface ใน Golang ซะส่วนใหญ่
* Keep the contract simple.

***

### GithubGist

[GithubGist](https://gist.github.com/alferov/e85863cbaa2143308a25d24bdb101833)

#### การให้ความหมาย

* This principle states that we should be able to replace a class in a program with another class as long as both classes implement the same interface. After replacing the class no other changes should be required and the program should continue to work as it did originally.

***

> \[opinion] ถึงตรงนี้เข้าใจว่า ปัญหาจะมี 3 ลักษณะคือ
>
> * การ design Supertype ที่ไม่ดี ทำให้บางทีก็ implement method ไม่ได้ หรือ implement แล้วทำให้เกิดพฤติกรรมที่ไม่พึงประสงค์
> * ตัวภาษา เช่น Dynamic language จะทำให้มีโอกาสที่ return type ไม่เหมือนกัน หรือ ไม่ได้ Overide method ของ parent ใน Java
> * พฤติกรรมของการ implement ต่างกัน ทำให้มี side effect ต่างกัน เช่น การ throw exception

### Further reading

พอเริ่มเข้าใจก็ขี้เกียจอ่านต่อ แปะไว้ละกัน

* [Baeldung](https://www.baeldung.com/java-liskov-substitution-principle)
  * Describes from Open-Closed Principle to Liskov Substitution Principle.
  * Describes why breaking LSP will break OCP in consequence.
  * Describes code-smell.
* [alpharithms](https://www.alpharithms.com/liskov-substitution-principle-lsp-solid-114908)
  * Don't know what they tell.
* [s8sg](https://s8sg.medium.com/solid-principle-in-go-e1a624290346)
  * Describes with examples in Go.
* [levelup](https://levelup.gitconnected.com/practical-solid-in-golang-liskov-substitution-principle-e0d2eb9dd39)
  * Other Go examples.
* [solid-development-principles-in-motivational-pictures](http://web.archive.org/web/20160521015258/https://lostechies.com/derickbailey/2009/02/11/solid-development-principles-in-motivational-pictures)
  * Memes.
