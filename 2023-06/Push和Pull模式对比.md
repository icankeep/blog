## 参考博客
https://cloud.tencent.com/document/product/406/4791

## 总结

- Push模式实时性更高，但消费者无法根据自身消费情况调整，且当消费者离线时，Server端需要对消息持久化，消费者再上线时，需要重新推消息
Server端逻辑复杂

- Pull模式消费者可以根据自身维护的位移去Server端拉取消息，可以根据自身负载情况调整速率，不过如果Pull过于频繁，可能给Server带来额外压力，
如果间隔拉长，则消息会有一定延迟。一般可能会采用长轮训的方式代替简单的Pull