# RXJAVA

[toc]

reference: 

https://medium.com/gradeup/simplest-rxjava-tutorial-for-android-262013ce3545

```java
Observable{
     //The work you need to do
}
.subscribeOn(Schedulers.io) //thread you need the work to perform on
.observeOn(AndroidSchedulers.mainThread()) //thread you need to handle the result on
.subscribeWith(Observer{
     //handle the result here
})
```

