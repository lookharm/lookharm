# Go: The evolution of sync/mutex \[Part 1]

ℹ️ This article is based on Go 1.17.

&#x20;   Mutex หนึ่งในเครื่องมือหรือกลวิธีที่ทำให้โกรูทีน (goroutine) มีการทำงานที่สอดประสานกัน เช่น การอ่านและการเขียนค่าของตัวแปร เป็นต้น Mutex มีการใช้งานที่เรียบง่ายโดยเรียกใช้ฟังก์ชัน Lock และ Unlock แต่หากเราลองดูซอร์สโค้ดใน sync/mutex ของ Go1.17 \[1] ก็จะเห็นรายละเอียดที่ค่อนข้างมากประมาณ 135 บรรทัด \[2] (ไม่รวมคอมเมนต์) มีทั้งการแบ่งออกเป็น 2 โหมด stavation และ normal, การทำ spin, การทำ bit shift เพื่อใช้นับจำนวน watier ฯลฯ\
บทความนี้เรามาลองทำความเข้าใจเบื้องหลังการทำงานของ Mutex ในแต่ละส่วน แต่หากจะเริ่มต้นที่ Go1.17 ในปี 2022 นี้ดูแล้วจะยากไป เพื่อความง่ายเราจะย้อนเวลากลับไปเมื่อประมาณ 14 ปีที่แล้วในตอนที่ Mutex ถูกเขียนขึ้นเป็นครั้งแรก

***

<details>

<summary>ซอร์สโค้ด: sync/mutex (Go1.17)</summary>

```go
package sync

import (
	"internal/race"
	"sync/atomic"
	"unsafe"
)

type Mutex struct {
	state int32
	sema  uint32
}

type Locker interface {
	Lock()
	Unlock()
}

const (
	mutexWoken
	mutexStarving
	mutexWaiterShift = iota

	starvationThresholdNs = 1e6
)

func (m *Mutex) Lock() {
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	m.lockSlow()
}

func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false
	awoke := false
	iter := 0
	old := m.state
	for {
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state
			continue
		}
		new := old
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		if awoke {
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken
		}
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			if old&(mutexLocked|mutexStarving) == 0 {
			}
			queueLifo := waitStartTime != 0
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
			if old&mutexStarving != 0 {
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					throw("sync: inconsistent mutex state")
				}
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving
				}
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0
		} else {
			old = m.state
		}
	}

	if race.Enabled {
		race.Acquire(unsafe.Pointer(m))
	}
}

func (m *Mutex) Unlock() {
	if race.Enabled {
		_ = m.state
		race.Release(unsafe.Pointer(m))
	}

	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		m.unlockSlow(new)
	}
}

func (m *Mutex) unlockSlow(new int32) {
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}
	if new&mutexStarving == 0 {
		old := new
		for {
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else {
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```

</details>

***

## add mutex.Mutex (bf3d)

```
commit bf3dd3f0efe5b45947a991e22660c62d4ce6b671
Author: Russ Cox <rsc@golang.org>
Date:   Thu Dec 4 12:51:36 2008 -0800

    add mutex.Mutex
    
    R=r
    DELTA=349  (348 added, 0 deleted, 1 changed)
    OCL=20380
    CL=20472
```

&#x20;    เมื่อวันที่ 4 ธันวาคม ค.ศ. 2008 ก่อนที่ Go1 จะถูกปล่อยออกมา Russ Cox หนึ่งในผู้พัฒนาหลักของ Go ได้ทำการ commit ด้วยข้อความที่แสนเรียบง่ายว่า `"add mutex.Mutex"` \[3] และนี่คือจุดเริ่มต้นของ Mutex หากลองดูซอร์สโค้ด \[4] ก็จะพบว่าเป็นเวอร์ชั่นที่เรียบง่ายนับเป็นบรรทัดโดยประมาณ 32 บรรทัด (ไม่รวมคอมเมนต์) \[5]

***

<details>

<summary>ซอร์สโค้ด: sync/mutex (commit bf3d)</summary>

```go
package sync

package func cas(val *int32, old, new int32) bool

export type Mutex struct {
	key int32;
	sema int32;
}

func xadd(val *int32, delta int32) (new int32) {
	for {
		v := *val;
		if cas(val, v, v+delta) {
			return v+delta;
		}
	}
	panic("unreached")
}

func (m *Mutex) Lock() {
	if xadd(&m.key, 1) == 1 {
		return;
	}
	sys.semacquire(&m.sema);
}

func (m *Mutex) Unlock() {
	if xadd(&m.key, -1) == 0 {
		return;
	}
	sys.semrelease(&m.sema);
}
```

</details>

***

การทำงานในแต่ละส่วน

### Mutex struct

```go
export type Mutex struct {
	key int32;
	sema int32;
}
```

* Mutex เป็น struct ที่มี method สองตัวชื่อ Lock และ Unlock
* key บอกสถานะของ Mutex ว่าล็อคอยู่หรือไม่ และเก็บจำนวน waiter
* key > 0 คือ Mutex โดนล็อคอยู่
* sema เก็บสถานะที่ใช้กับฟังก์ชัน semaacquire และ semarelease

> note:
>
> * key และ sema เป็น int32 จึงทำให้สามารถติดลบได้
> * Go ในตอนเริ่มแรกมีคีเวิร์ดชื่อ `export` และเซมิโคลอนด้วยเช่นกัน แต่หลังจาก Go1 ก็ไม่มีให้เห็นแล้ว

ก่อนจะเข้าสู่ฟังก์ชัน Lock และ Unlock มี 4 ฟังก์ชันที่น่าสนใจและจำเป็นต้องรู้ก่อนได้แก่ cas, xadd, semaacquire และ semarelease

### cas

```go
package func cas(val *int32, old, new int32) bool
```

* cas หรือ compare-and-swap เป็น atomic instruction \[6]
* `func cas` ในที่นี้จะเป็นตัวอ้างอิงไปยัง cas ที่ implement ด้วย assembly อีกที \[7]
*   เพื่อให้เข้าใจวิธีการทำงานของ cas มากขึ้น ลองดูคอมเมนต์ในซอร์สโค้ด ที่ Russ Cox เขียนไว้

    ```
    // bool cas(int32 *val, int32 old, int32 new)
    // Atomically:
    //	if(*val == old){
    //		*val = new;
    //		return 1;
    //	}else
    //		return 0;
    TEXT cas(SB), 7, $0
    ```
*   ตัวอย่าง

    ```go
    x := 10
    cas(&x, 10, 15)     // 10 == 10
    fmt.Print(x)        // x = 15
    cas(&x, 100, 1)     // 15 != 100
    fmt.Print(x)        // x = 15
    ```

### xadd

```go
func xadd(val *int32, delta int32) (new int32) {
	for {
		v := *val;
		if cas(val, v, v+delta) {
			return v+delta;
		}
	}
	panic("unreached")
}
```

* xadd หรือ fetch-and-add \[8]
* เปลี่ยนแปลงค่า val ด้วยจำนวน delta ที่กำหนด
* เนื่องจาก cas อาจทำสำเร็จหรือไม่สำเร็จก็ได้ เพราะมีหลายๆ โกรูทีนที่แย่งกันเรียกใช้ cas จึงต้องวนลูปเพื่อการันตีว่าท้ายที่สุดแล้ว xadd จะเพิ่มค่าเข้าไปตามที่ผู้ใช้กำหนดได้สำเร็จ

### semaphore

* semaphore ในที่นี้อาจไม่ใช่ semaphore ตามนิยามของ Wiki โดยตรง \[9] แต่เป็น semaphore ที่เอาไว้ใช้สำหรับจัดการ การ sleep และ wakeup ของโกรูทีน \[10] คล้ายๆ Linux's Futex \[11]
* semaphore ในตอนเริ่มแรกใช้ linked-list และมีแนวโน้มว่าจะเปลี่ยนเป็น hash table of linked-list
* `semaacquire(sema *int32)`: เข้าคิว (queue) sleep รอจนกว่า sema > 0 จากนั้นลดค่า sema ลง 1
* `semrelease(sema *int32)`: เพิ่มค่า sema ขึ้น 1 และ wake โกรูทีนที่อยู่ในคิว

หลังจากรู้จักฟังก์ชันที่น่าสนใจกันไปแล้วมาดูฟังก์ชัน Lock และ Unlock ต่อ

### Lock

* เรียก xadd  โดยปรับค่าตัวแปร key ให้เพิ่มขึ้น 1
* ถ้าไม่เกิด contention: key เปลี่ยนจาก 0 ไปเป็น 1 ก็สามารถล็อค (lock) ได้เลย
* ถ้าเกิด contention: (sleep) key มีค่ามากว่า 1 ก็จะเรียก sys.semacquire เพื่อให้โกรูทีนเข้าคิวรอ กล่าวคือการทำงานของโค้ดจะรออยู่ที่บรรทัดที่เรียก sys.semaacquire นี้จนกว่าจะมีโกรูทีนตัวอื่นไปเรียก sys.semrelease

### Unlock

* เรียก xadd เช่นกัน แต่ส่ง -1 เพื่อปรับค่าตัวแปร key ให้มีค่าลดลง 1
* ถ้าไม่เกิด contention: key เปลี่ยนจาก 1 ไปเป็น 0 ก็สามารถปลดล็อค (unlock) ได้เลย
* ถ้าเกิด contention: (wake) key มีค่ามากว่า 1 แสดงว่ามีโกรูทีนรออยู่ในคิวก็จะเรียก sys.semrelease เพื่อส่งสัญญาณไปยังโกรูทีนที่รออยู่ในคิวให้ออกมาทำงานได้

> note:\
> ใน version นี้ไม่มีการป้องกัน Unlock Mutex ที่ยังไม่ได้ล็อค&#x20;
>
> จะเห็นว่าจะมีกรณีที่ key เป็น -1 ได้ แต่ทางทีมพัฒนาไม่ได้เผื่อกรณีนี้ไว้ในตอนแรก

***

## การเปลี่ยนแปลงของ Mutex จาก commit bf3d จนถึง Go1.17

&#x20;   โค้ดของ Mutex นั้นมีการเปลี่ยนแปลงอยู่ตลอด \[12] \[13] มีทั้งที่ทำโดยตรงกับ Mutex และส่วนที่เป็น low level แต่ส่งผลกระทบต่อ Mutex แต่การเปลี่ยนแปลงครั้งใหญ่ที่มีผลต่อโค้ดของ Mutex โดยตรงหากนับจาก commit bf3d (add mutex.Mutex) จนถึง Go1.17 จะมีประมาณ 4 commit ด้วยกันซึ่งสามารถแบ่งได้เป็น 2 ช่วงคือ ก่อนปล่อย Go1 และ หลังปล่อย Go1

ก่อน Go1:

1. "add mutex.Mutex" by Russ Cox \[3] (bf3dd3f0efe5b45947a991e22660c62d4ce6b671)
2. "improve Mutex to allow successive acquisitions" by Dmitriy Vyukov \[14] (dd2074c82acda9b50896bf29569ba290a0d13b03)

หลัง Go1:

1. "sync: add active spinning to Mutex" by Dmitriy Vyukov \[15] (edcad8639a902741dc49f77d000ed62b0cc6956f)
2. "sync: make Mutex more fair" by Dmitriy Vyukov \[16] (0556e26273f704db73df9e7c4c3d2e8434dec7be)

***

## หลังจาก "add mutex.Mutex" จนถึงก่อน "improve Mutex to allow successive acquisitions"

&#x20;   ในระหว่างทางจาก commit แรกของ Mutex (bf3d) จนถึง commit ก่อนที่จะทำให้เกิดการเปลี่ยนแปลงใหญ่ครั้งที่ 2 (6a18) \[17] มีการเปลี่ยนแปลงเล็กน้อยเกิดขึ้นอยู่ตลอด มีส่วนที่น่าสนใจดังนี้

* ลบคีเวิร์ด export

```diff
-export type Mutex struct {
+type Mutex struct {
```

* เพิ่ม RWLock

```diff
+type RWMutex struct {
+	m Mutex;
+}
```

* ย้ายไดเร็กทอรีจาก src/lib ไปเป็น src/pkg
* ปรับฟอร์แหม็ทของโค้ดด้วย gofmt
* ปรับปรุงคอมเมนต์
* เพิ่มการตรวจสอบการเรียก Unlock ของ Mutex ที่ไม่ได้ล็อคอยู่

```diff
-	if xadd(&m.key, -1) == 0 {
+	switch v := xadd(&m.key, -1); {
+	case v == 0:
 		// changed from 1 to 0; no contention
 		return
+	case int32(v) == -1:
+		// changed from 0 to -1: wasn't locked
+		// (or there are 4 billion goroutines waiting)
+		panic("sync: unlock of unlocked mutex")
```

* ใช้ sync/atomic แทน xadd

```diff
-	if xadd(&m.key, 1) == 1 {
+	if atomic.AddInt32(&m.key, 1) == 1 {
```

***

<details>

<summary>ซอร์สโค้ด: sync/mutex (commit 6a18)</summary>

```go
package sync

import (
	"runtime"
	"sync/atomic"
)

type Mutex struct {
	key  int32
	sema uint32
}

type Locker interface {
	Lock()
	Unlock()
}

func (m *Mutex) Lock() {
	if atomic.AddInt32(&m.key, 1) == 1 {
		return
	}
	runtime.Semacquire(&m.sema)
}

func (m *Mutex) Unlock() {
	switch v := atomic.AddInt32(&m.key, -1); {
	case v == 0:
		return
	case v == -1:
		panic("sync: unlock of unlocked mutex")
	}
	runtime.Semrelease(&m.sema)
}
```

</details>

***

## จบพาร์ทแรก

* สิ่งที่ผู้อ่านน่าจะได้รู้จักเพิ่มเติม
  * semaphore ในแบบของ Go
  * atomic insturction: compare-and-swap (cas), fetch-and-add (xadd)
  * ความตั้งใจแรกของการ implement Mutex ของ Go คือการใช้ semaphore (คิว) เพื่อให้เกิด fairness ใครมาก่อนก็ได้ Lock ก่อน
  * Lock ไม่ได้ผูกอยู่กับโกรูทีนตัวใดตัวหนึ่ง Mutex โดนล็อคโดยโกรูทีนตัวใดตัวหนึ่งและโดนปลดล็อคโดยโกรูทีนไหนก็ได้\
    (ตรงนี้มีประเด็นที่ทำให้ถกเถียงกันอยู่ในภายหลัง \[18])
  * Mutex เป็นเพียงตัวแปร struct ตัวนึงเท่านั้น
* โค้ดที่เพิ่มขึ้นมาในภายหลังทำไปเพื่อแก้ปัญหาที่พบเจอหรือเพิ่มประสิทธิภาพการทำงาน จากที่ดูเรียบง่ายก็ซับซ้อนขึ้นมา
* ในส่วนของพาร์ทถัดไปจะกล่างถึง "improve Mutex to allow successive acquisitions"

***

## References

* \[1] [Mutex Go1.17](https://github.com/golang/go/blob/release-branch.go1.17/src/sync/mutex.go)
* \[2] [Mutex Go1.17 (no comment)](https://gist.github.com/lookharm/e5ac7aba9e15cc2e0ae38507cd57d7e3)
* \[3] [commit bf3d "add mutex.Mutex"](https://github.com/golang/go/commit/bf3dd3f0efe5b45947a991e22660c62d4ce6b671)
* \[4] [Mutex bf3d implementation](https://github.com/golang/go/blob/bf3dd3f0efe5b45947a991e22660c62d4ce6b671/src/lib/sync/mutex.go)
* \[5] [Mutex bf3d (no comment)](https://gist.github.com/lookharm/8158b978bd1dc1aac1802b60ddd5094b)
* \[6] [Compare-and-swap (Wiki)](https://en.wikipedia.org/wiki/Compare-and-swap)
* \[7] [Compare-and-swap bf3d implementation](https://github.com/golang/go/blob/bf3dd3f0efe5b45947a991e22660c62d4ce6b671/src/lib/sync/asm\_amd64.s)
* \[8] [Fech-and-add: xadd (Wiki)](https://en.wikipedia.org/wiki/Fetch-and-add)
* \[9] [Semaphore (Wiki)](https://en.wikipedia.org/wiki/Semaphore\_\(programming\))
* \[10] [Semaphore bf3d impementation](https://github.com/golang/go/blob/bf3dd3f0efe5b45947a991e22660c62d4ce6b671/src/runtime/sema.c)
* \[11] [Linux's Futex (Wiki)](https://en.wikipedia.org/wiki/Futex)
* \[12] [Mutex diff before release Go1](https://gist.github.com/lookharm/534c7587075881bd4cfc0674c84e0b76)
* \[13] [Mutex diff after release Go1](https://gist.github.com/lookharm/7f191c3a2e962217b9b3e1cc7312714a)
* \[14] [commit dd20 "sync: improve Mutex to allow successive acquisitions"](https://github.com/golang/go/commit/dd2074c82acda9b50896bf29569ba290a0d13b03)
* \[15] [commit edca "sync: add active spinning to Mutex"](https://github.com/golang/go/commit/edcad8639a902741dc49f77d000ed62b0cc6956f)
* \[16] [commit 0556 "sync: make Mutex more fair"](https://github.com/golang/go/commit/0556e26273f704db73df9e7c4c3d2e8434dec7be)
* \[17] [commit 6a18 "src/pkg: make package doc comments consistently start with "Package f…"](https://github.com/golang/go/commit/6a186d38d1c7e98487378991dba3085acbdb05c2)
* \[18] [proposal: sync: prohibit unlocking mutex in a different goroutine #9201](https://github.com/golang/go/issues/9201)
