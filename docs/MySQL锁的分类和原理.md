# MySQL锁的分类和原理

## 锁是什么？
通俗的说就是你去蹲坑的时候要把厕所的门从里面反锁一下，告诉别人这个坑位已经有人在使用了，他们需要等等才能使用。

## 锁的优缺点
在多线程并发访问的时候，锁可以保证资源的完整性，但会牺牲一定的系统开销。

## 锁的分类
![](https://wx3.sinaimg.cn/large/0081fa71ly1gmcne9lrdcj30hl0d6jrh.jpg)

- 乐观锁
- 悲观锁
    - 共享锁
    - 排它锁
        - 行锁
        - 间隙锁
        - 表锁
    
### 乐观锁
顾名思义，就是比较乐观的锁，在 // TODO