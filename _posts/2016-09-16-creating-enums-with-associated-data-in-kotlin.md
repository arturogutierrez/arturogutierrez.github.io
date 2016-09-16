---
layout: post
title: Creating enums with associated data in Kotlin
comments: true
---

Lately, I've been developing in Swift due to job reasons and I love the [Swift's enum with associated values][swiftenums]. If you don't know them I'll show you an example but, I can tell you something, you will love them too!.

Usually, an `enum` can contain associated data but the type of that data should be the same for all enum cases. Example:

```java
public enum DayOfWeek {
   MONDAY(1),
   TUESDAY(2),
   WEDNESDAY(3),
   THURSDAY(4),
   FRIDAY(5),
   SATURDAY(6),
   SUNDAY(7);

   private int dayNumber;
   private DayOfWeek(int dayNumber) {
      this.dayNumber = dayNumber;
   }
   public int getDayNumber() {
      return dayNumber;
   }
}
```

Same code in Kotlin (much more beautiful):

```kotlin
enum class DayOfWeek(val dayNumber: Int) {
  MONDAY(1), TUESDAY(2), WEDNESDAY(3), THURSDAY(4),
  FRIDAY(5), SATURDAY(6), SUNDAY(7)
}
```

Standard Java enums can't support a different kind of associated values per case so this means that we can't have something like:

```java
   MONDAY(1),
   TUESDAY(2),
   WEDNESDAY(3),
   THURSDAY(4),
   FRIDAY(5),
   SATURDAY("WEEKEND"),
   SUNDAY("WEEKEND");
```

I know, the example is a little bit weird but you got the enum's associated value limitation, right?

Now, imagine that you are in charge of the development of a new instant messaging app and you are designing the model of all type events that a conversation can have:

* Texts
* Images
* Audios

Obviously, there are some common fields in all cases:

* Event date
* If it's incoming or outgoing event
* Delivery status, etc.

Maybe, you first model approach could create a base class named `Event` and start to create subclasses for each kind of event, that's ok, I would do the same thing probably, but, what if I tell you that there is a better way (IMHO) to do it using enums with different associated data? cool, right?.

The idea is to use the powerful of the enum's associated values but the sad part is we can't do it directly with enums in Kotlin because they have the same limitations than Java but don't worry, there is something new called [Sealed classes][sealedclasses] in Kotlin that emulates this directly, in fact, we can say they are the replacement for enums.

Instead of explaining about what is [Sealed classes][sealedclasses] we will start with an example, exactly with the `DeliveryStatus` field. We want to store the delivery status of an event so we have three possibilities:

* Delivered
* Delivering
* NotDelivered. This is special because we also want to store the error message to show the user about why the event couldn't be delivered.

Using a sealed class named `DeliveryStatus` we can model it this way:

```kotlin
sealed class DeliveryStatus {
    class Delivered : DeliveryStatus()
    class Delivering : DeliveryStatus()
    class NotDelivered(val error: String) : DeliveryStatus()
}
```

This sealed class acts as an enum really because they can't be extended and each "internal class" acts an enum case, this way you can define the associated data per possibility individually. You can read more (not much more) about them in the [Kotlin Docs][sealedclasses].

Next one, event direction:

* Incoming. Should include who sent the event.
* Outgoing. Should include the previous delivery status

```kotlin
sealed class Direction {
    class Incoming(val from: String) : Direction()
    class Outgoing(val status: DeliveryStatus) : Direction()
}
```

Now, we'll model the event content type, remember could be three kinds of events: Text, Audio, and Image.

```kotlin
sealed class ContentType {
    class Text(val body: String) : ContentType()
    class Image(val url: String, val caption: String) : ContentType()
    class Audio(val url: String, val duration: Int) : ContentType()
}
```

Notice that each kind of event has its own type of associated data: a body for messages, URL and caption for images, and URL and duration for audios.

Finally, we need the final data class which defines the content of a full event:

```kotlin
data class Event (val contentType: ContentType, val direction: Direction)
```

I'm going to show you an example of how to use them:

```kotlin
// An event list example but this could be retrieved from a Conversation data store in a real app
val events = listOf(
    Event(ContentType.Text("Hey, I'm Tony"), Direction.Incoming("Tony Stark")),
    Event(ContentType.Image("URL_TO_IMAGE", "Avengers"),  Direction.Incoming("Bruce Banner")),
    Event(ContentType.Audio("URL_TO_AUDIO", 15), Direction.Outgoing(DeliveryStatus.Delivered()))
)

for(event in events) {
    renderEvent(event)
}

fun renderEvent(event: Event): Unit {
    when(event.contentType) {
        is ContentType.Text -> println("${event.contentType.body}")
        is ContentType.Audio -> println("Audio of ${event.contentType.duration} secs.")
        is ContentType.Image -> println("Image (${event.contentType.caption})")
    }
}
```
Notice that `event.contentType` is casted inside of the `is` clause, this allow us to access to specific fields being compile safety.

That's all, we have modeled our events using sealed class in a fashion way, but, obviously give us more benefits:

* The compiler knows everything about all cases and the `when` statement is able to handle all possible cases at compiler time.
* All events are inmutables
* Everything is more clear and readable

If you are interested you can **play with all source code** in [Try Kotlin][example].

Finally, I can say you we're using this kind of events model in our iOS project written in Swift and we're very happy with the result, make everything more concise, readable and beautiful, so the ability to have something similar in Kotlin is awesome!.

<a href="https://twitter.com/share" class="twitter-share-button" data-url="http://arturogutierrez.com{{ page.url }}" data-via="{{ site.twitter.username }}" data-size="large">Tweet</a>

<!-- Put this just before the closing body tag -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>


[swiftenums]: https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html
[sealedclasses]: https://kotlinlang.org/docs/reference/classes.html#sealed-classes
[example]: http://try.kotlinlang.org/#/UserProjects/5pqf3jglalsp68m2gtppi13k19/b48uqn9onkngl8uh3qhmcihj9n
