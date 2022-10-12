---
title: "DB Transaction"
excerpt: "DB 트랜잭션 개념 정리"
categories:
  - database
tags:
  - transaction
toc: true
---

- [1. 정의](#1-정의)
- [2. 특징 (ACID)](#2-특징-acid)
- [3. 트랜잭션 제어 명령](#3-트랜잭션-제어-명령)
	- [3.1. Commit](#31-commit)
	- [3.2. Rollback](#32-rollback)
- [4. 트랜잭션 라이프 사이클](#4-트랜잭션-라이프-사이클)
- [5. 트랜잭션 이용 시 주의사항](#5-트랜잭션-이용-시-주의사항)
- [6. 참고](#6-참고)

---

<br>

# 1. 정의

- 데이터베이스의 상태를 변경하기 위해 하나의 논리적 기능을 수행하기 위한 작업의 단위 혹은 한꺼번에 수행되어야할 일련의 연산

<br><br>

# 2. 특징 (ACID)

| 특징 | 설명 | 예시 |
| --- | --- | --- |
| Atomicity(원자성) | - 더 작게 나눌 수 없는 최소의 단위라는 의미 <br> - 트랜잭션이 데이터베이스에 모두 반영되거나, 하나도 반영되지 않는 것을 보장 | 계좌이체 도중 입금처리 실패 시 출금된 돈을 다시 입금처리해서 이전상태를 유지 |
| Consistency(일관성) | - 트랜잭션 처리 결과가 항상 일관적임을 보장 <br> - 데이터 무결성 제약조건을 보장 <br> - 트랜잭션이 DB를 비정상적인 상태로 변경하려하면 해당 트랜잭션을 취소하고 이전의 정상적인 상태로 롤백하는 것을 보장 | 모든 고객 계좌의 잔액은 양수여야한다는 제약조건이 명시되어 있을때, 고객계좌의 잔액을 음수로 만드는 트랜잭션은 롤백된다. |
| Isolation(독립성) | - 수행 중인 트랜잭션은 완료 이전까지 다른 트랜잭션의 수행 결과를 참조하지 않는 것을 보장 <br> - 수행 중인 트랜잭션에 다른 트랜잭션의 연산작업이 끼어들어 기존 작업에 영향을 주지 않는 것을 보장 | 계좌이체 작업을 진행하는 도중에 해당 계좌의 잔액을 조회한다거나 하는 작업을 동시에 수행할 수 없다 |
| Durability(지속성) | - 트랜잭션이 성공적으로 완료되면, 그 결과는 영구적으로 반영되어야 한다. | 은행시스템 장애 발생 후 복구되어도 계좌의 금액은 기존과 같아야한다. |

<br><br>

# 3. 트랜잭션 제어 명령

## 3.1. Commit

- 하나의 트랜잭션이 성공적으로 끝났고, 데이터베이스가 일관성있는 상태에 있을 때 하나의 트랜잭션이 끝났음을 알려주기 위해 사용하는 명령
- 트랜잭션이 수행한 변경사항을 저장하는 명령
    - 직전 commit이나 rollback부터 현재(commit 명령 시점)까지의 변경사항

## 3.2. Rollback

- 하나의 트랜잭션이 비정상적으로 종료되어 데이터베이스의 일관성을 깨뜨렸을 때, 해당 트랜잭션의 일부가 정상적으로 처리되었더라도 트랜잭션의 원자성을 구현하기 위해 해당 트랜잭션이 행한 모든 연산을 취소(Undo)하는 명령
- 트랜잭션이 수행한 변경사항을 되돌리는 명령
    - 직전 commit이나 rollback부터 현재(rollback 명령 시점)까지의 변경사항

<br><br>

# 4. 트랜잭션 라이프 사이클

<img style="background-color: white" alt="transaction_life_cycle.png" src="../images/transaction_life_cycle.png">

<br>

| 상태 | 설명 |
| --- | --- |
| Active(활동) | - 트랜잭션 실행 후 첫 번째 상태 <br> - 트랜잭션이 실행중인 상태 <br> - 읽기/쓰기 연산을 수행하는 중에는 활동상태 유지 |
| Partially Committed(부분완료) | - 트랜잭션의 변경사항을 위한 연산이 모두 수행되었고, 아직 변경사항이 디스크에 반영(commit)되지 않은 상태 <br> - 부분완료 상태에서 변경된 데이터는 메모리 버퍼에만 저장되어있음 |
| Committed(완료) | - 트랜잭션이 정상적으로 완료된 상태 <br> - 트랜잭션의 변경사항이 DB에 영구적으로 기록된 상태 |
| Failed(실패) | - 트랜잭션이 정상적으로 진행될 수 없는 상태 <br> - 활동 혹은 부분완료 상태에서 트랜잭션이 실패하거나 취소되면 실패상태로 변경됨 |
| Terminated(종료) | 완료 혹은 취소 이후 트랜잭션의 마지막 상태 |

<br><br>

# 5. 트랜잭션 이용 시 주의사항

- 트랜잭션 범위 최소화할 것
    - 제한된 DB 커넥션을 하나의 트랜잭션이 소유하는 시간이 길어지면, 여유 커넥션의 개수가 줄어들고 대기시간이 늘어날 수 있다.

<br><br>

# 6. 참고

[https://fauna.com/blog/database-transaction](https://fauna.com/blog/database-transaction)

[https://www.bmc.com/blogs/acid-atomic-consistent-isolated-durable/](https://www.bmc.com/blogs/acid-atomic-consistent-isolated-durable/)

[https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback](https://www.digitalocean.com/community/tutorials/sql-commit-sql-rollback)