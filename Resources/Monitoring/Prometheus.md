# Instant Vector vs Range Vector

* [Decoding PromQL: Unraveling Range Vectors and Instant Vectors in Prometheus](https://medium.com/@ahmed.s.farag96/decoding-promql-unraveling-range-vectors-and-instant-vectors-in-prometheus-c1390f650e5c)


![[Pasted image 20240114171848.png | white]] 
* Time Series : 시간 간격으로 배치된 데이터들의 수열 (시계열)
* Sample : Time Series 내 한 개의 데이터
* Metric : 
* Label : 
* Time Series Identifier : Metfic + Label[]


![[Pasted image 20240114172855.png | white]]
![[Pasted image 20240114173027.png | white]]
* Instant Vector : metrics values within ***the instant time*** (***one sample*** of each time series)
* Range Vector : metrics values within the ***time range*** (***multiple sample*** of each time series)