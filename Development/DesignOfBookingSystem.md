# How to design a booking system to avoid overlapping reservation
https://medium.com/@oleg0potapov/how-to-design-a-booking-system-to-avoid-overlapping-reservation-fe17194c1337

- A: 2월 6일부터 2월 9일까지 예약
- B: 2월 5일부터 2월 8일까지 예약

<img width="552" alt="스크린샷 2023-02-04 17 31 31" src="https://user-images.githubusercontent.com/19607962/216757474-9887801c-ea6c-4f89-8a49-ff9b4617348b.png">


두 명의 유저가 각각 다른 기간으로 예약을 진행하는 경우, 어떻게 처리할 것인지에 대해 이야기합니다.

한 명만 예약이 정상처리되고, 나머지 유저는 실패처리되어야 합니다.

## 아무처리도 하지 않았을 경우

A,B 가 동시에 요청했을 경우, 둘 다 예약이 가능한 경우가 발생할 수 있습니다. (DB 레벨에서 특별한 처리를 안 했을 경우)

## UniqueIndex?

‘**CREATE UNIQUE INDEX idx_bookings_dates ON bookings (room_id, start_date, end_date);**’

위와 같은 유니크인덱스를 추가했을 때, (start_date, end_date) 가 같은 경우에만 중복처리를 방지할 수 있으므로

기존의 예시인 아래 경우는 제대로 처리되지 않는 문제가 발생합니다.

> A: 2월 6일부터 2월 9일까지 예약
> B: 2월 5일부터 2월 8일까지 예약




