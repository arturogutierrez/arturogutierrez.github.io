---
layout: post
title: Using RxJava to create a data store with hot observables
comments: true
---

I've always wanted to learn [RxJava][rxjava] so I said myself to start to using it in the next pet project, and that is what I did in my latest project: [Openticator][openticator].

I found the first problem working with [RxJava][rxjava] when I was coding a simple CRUD data store: **how can I detect changes occurred in the data store?**

If you are not familiarized about what is data store we can say that *it's a repository for persistently storing and managing collections of data which include not just repositories like databases but also simpler store types such as simple files, emails etc* [Wikipedia][datastore].

This is how my account data store looks like:

```java
public interface AccountDataStore {

  Observable<Account> add(Account account);

  Observable<Account> update(Account account);

  Observable<Void> remove(Account account);

  Observable<List<Account>> getAllAccounts();
}
```

You can see that all operations are [Observables][observable] in order to execute in background leaving the main thread free and, also, chaining them with other operations that are beside the point.

## Hot Observable

What I want to achieve is convert the `getAllAccounts` [Observable][observable] in a hot observable. A hot observable is a sequence that is active and produces notifications regardless of subscriptions.

The idea is **emitting the entire account list whenever there is a change** in any account.
You can say that sending the entire list each time it's not optimal (and probably you're right) but [Openticator][openticator] works with only few accounts so the performance is not critical here.

## Solution

[RxJava][rxjava] offers a abstract class `Subject` to create hot observables. From the docs, a `Subject` represents an object that **is both an Observable and an Observer**.

First thing I have to do is initialize the `Subject` in the constructor:

```java
  private final Subject<Void, Void> changesPublishSubject;

  @Inject
  public AccountDiskDataStore() {
    this.changesPublishSubject = PublishSubject.<Void>create().toSerialized();
  }
```
`PublishSubject` is an implementation from `Subject` which emits all subsequently observed items to the subscriber. The idea is **publishing a `null` value** into this Subject whenever there is a change in the data store.
You can ask me **Why publish a null value?** The reason is I only want to detect whenever there is a change and I don't need to know what was changed, so a *null value* is enough for this case.
Also, I created it as **serialized** because I don't want to be worried about from which thread all null values are being sent to the subject, to be more specific the official docs says about `toSerialized()`: *Wraps a Subject so that it is safe to call its various on methods from different threads* [Docs][http://reactivex.io/RxJava/javadoc/rx/subjects/Subject.html#toSerialized()].

The data store should emit the account list when there is a new value (*null value*) in `changesPublishSubject`, so the first implementation could be:

```java
  @Override
  public Observable<List<Account>> getAllAccounts() {
    return changesPublishSubject.map(aVoid -> getAccountsAsBlocking());
  }
```
*Assume that `getAccountsAsBlocking` is retrieving all accounts from disk synchronously.*

The idea is map each `null` value received from the `Subject` observable into a new fresh retrieved list. And it works!, it emits the entire account list when it's necessary but there is a problem: **no list is being emitted immediately when a new observer subscribes** being blocked. No problem, it's easily fixed providing an initial value to the observable using `startsWith`:

```java
  @Override
  public Observable<List<Account>> getAllAccounts() {
    return changesPublishSubject.map(aVoid -> getAccountsAsBlocking())
        .startWith(Observable.fromCallable(this::getAccountsAsBlocking));
  }
```

Only one last thing is missing, telling to `changesPublishSubject` that there has been a change. It can be done calling `changesPublishSubject.onNext(null)` after any modifying operation. Example:

```java
  @Override
  public Observable<Account> add(Account account) {
    Observable<Account> accountObservable = return Observable.fromCallable(() -> {
      // Creating account in disk storage
      // [...]

      // Returning new account created
      return account;
    });

    return accountObservable.doOnNext(accountAdded -> notifyAccountChanges());
  }

  private void notifyAccountChanges() {
    changesPublishSubject.onNext(null);
  }  
```

The key thing is calling `notifyAccountChanges` in the `doOnNext` of each data store operation. Here is an example when adding an account but same principle is applied when updating or removing an account.

From this point, any observer that subscribes to the `getAllAccounts` observable will receive the entire account list immediately and also whenever there is a change in any account.

If you are interested in the full solution you can find it in the [Openticator GitHub repository][openticator], or direct link to [AccountDiskDataStore.java][accountdiskdatastore].

If you have any questions or comments, please post them below.
If you liked this post, you can
<a href="https://twitter.com/intent/tweet?url=http://arturogutierrez.com{{ page.url }}&text={{ page.title }}&via={{ site.twitter.username }}" 
   target="_blank">
  share it with your followers</a> 
or 
<a href="https://twitter.com/{{ site.twitter.username }}">
  follow me on Twitter</a>!

<a href="https://twitter.com/share" class="twitter-share-button" data-url="http://arturogutierrez.com{{ page.url }}" data-via="{{ site.twitter.username }}" data-size="large">Tweet</a>

<!-- Put this just before the closing body tag -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>


[rxjava]: https://github.com/ReactiveX/RxJava
[openticator]: https://github.com/arturogutierrez/Openticator
[datastore]: https://en.wikipedia.org/wiki/Data_store
[toserialized]: http://reactivex.io/RxJava/javadoc/rx/subjects/Subject.html#toSerialized()
[observable]: https://github.com/ReactiveX/RxJava/wiki/Observable
[accountdiskdatastore]: https://github.com/arturogutierrez/Openticator/blob/master/data/src/main/java/com/arturogutierrez/openticator/storage/AccountDiskDataStore.java

[twitter]: https://twitter.com/{{ site.twitter.username }}
[github]: https://github.com/{{ site.github.username }}
