---
layout: post
title: 100 Days Of SwiftUI - Aug 17 - 23
categories:
    - project
excerpt_separator:  <!--more-->
---

## [Day 59 - Project 12, part 3](https://www.hackingwithswift.com/100/swiftui/59)

One of the best ways to learn is to write your own code as often as possible, so here are three ways you should try extending this app to make sure you fully understand what’s going on.

All three of these tasks require you to modify the **`FilteredList`** view we made:

- Make it accept an array of **`NSSortDescriptor`** objects to get used in its fetch request.

    ```swift
    // In FilteredList initializer, add another argument to accept a list of NSSortDescriptor
    init(filterKey: String, filterValue: String, sortDescriptors: [NSSortDescriptor], @ViewBuilder content: @escaping (T) -> Content) {
        fetchRequest = FetchRequest<T>(entity: T.entity(), sortDescriptors: sortDescriptors, predicate: NSPredicate(format: "%K BEGINSWITH %@", filterKey, filterValue))
        self.content = content
    }

    // Create sortDescriptors constant and use it inside the FilteredList creation
    let sortDescriptors = [NSSortDescriptor(keyPath: \Singer.firstName, ascending: true)]
    FilteredList(filterKey: "lastName", filterValue: lastNameFilter, sortDescriptors: sortDescriptors) { (singer: Singer) in
        Text("\(singer.wrappedFirstName) \(singer.wrappedLastName)")
    }
    ```

- Make it accept a string parameter that controls which predicate is applied. You can use Swift’s string interpolation to place this in the predicate.

    ```swift
    // In FilteredList initializer, add another argumetn to accept predicate keyword
    init(filterKey: String, filterValue: String, sortDescriptors: [NSSortDescriptor], predicateKeyWord: String, @ViewBuilder content: @escaping (T) -> Content) {
        fetchRequest = FetchRequest<T>(entity: T.entity(), sortDescriptors: sortDescriptors, predicate: NSPredicate(format: "%K \(predicateKeyWord) %@", filterKey, filterValue))
        self.content = content
    }

    // In ContentView, create a new constant
    let predicateKeyWord = "BEGINSWITH"
    // Pass the predicateKeyWord to FilteredList
    FilteredList(filterKey: "lastName", filterValue: lastNameFilter, sortDescriptors: sortDescriptors, predicateKeyWord: predicateKeyWord) { (singer: Singer) in
        Text("\(singer.wrappedFirstName) \(singer.wrappedLastName)")
    }
    ```

- Modify the predicate string parameter to be an enum such as **`.beginsWith`**, then make that enum get resolved to a string inside the initializer.

    ```swift
    // Add PredicateKeyword
    enum PredicateKeyword: String, CustomStringConvertible {
        // Just some example of possible keywords
        // Find the whole list in https://nshipster.com/nspredicate/
        case beginsWith
        case contains
        case endsWith
        case like
        case matches
        
        var description: String { rawValue.uppercased() }
    }

    // Modify FilteredList to accept PredicateKeyword enum
    init(filterKey: String, filterValue: String, sortDescriptors: [NSSortDescriptor], predicateKeyWord: PredicateKeyword, @ViewBuilder content: @escaping (T) -> Content) {
        fetchRequest = FetchRequest<T>(entity: T.entity(), sortDescriptors: sortDescriptors, predicate: NSPredicate(format: "%K \(predicateKeyWord.description) %@", filterKey, filterValue))
        self.content = content
    }

    // Pass predicateKeyWord enum value to the constructor inside ContentView
    FilteredList(filterKey: "lastName", filterValue: lastNameFilter, sortDescriptors: sortDescriptors, predicateKeyWord: predicateKeyWord) { (singer: Singer) in
        Text("\(singer.wrappedFirstName) \(singer.wrappedLastName)")
    }
    ```

## [Day 58 - Project 12, part 2](https://www.hackingwithswift.com/100/swiftui/58)

Using `NSPredicate` to filter data stored in `Core Data`

In order to filter `Core Data` result. Inside a SwiftUI view, we can create a `FetchRequest` type, something like the following

`var fetchRequest = FetchRequest<Singer>`

Set the value to the property like `

```swift
fetchRequest = FetchRequest<Singer>(entity: Singer.entity(), sortDescriptors: [], predicate: NSPredicate(format: "lastName BEGINSWITH %@", "some string"))
```

Then we can simply fetch the value like `fetchRequest.lastName`

Also we can utilize generics to create SwiftUI views using the following code

```swift
struct FilteredList<T: NSManagedObject, Content: View>: View {
    var fetchRequest: FetchRequest<T>
    var singers: FetchedResults<T> { fetchRequest.wrappedValue }

    // this is our content closure; we'll call this once for each item in the list
    let content: (T) -> Content

    var body: some View {
        List(fetchRequest.wrappedValue, id: \.self) { singer in
            self.content(singer)
        }
    }

    init(filterKey: String, filterValue: String, @ViewBuilder content: @escaping (T) -> Content) {
        fetchRequest = FetchRequest<T>(entity: T.entity(), sortDescriptors: [], predicate: NSPredicate(format: "%K BEGINSWITH %@", filterKey, filterValue))
        self.content = content
    }
}
// We can get the view like this
FilteredList(filterKey: "lastName", filterValue: lastNameFilter) { (singer: Singer) in
    Text("\(singer.wrappedFirstName) \(singer.wrappedLastName)")
}
```

`Core Data` object relationship (one→one, or one→many) can be added to `xxx.xcdatamodeld` file. In order to make it work with SwiftUI, create a `NSManagedObject` subclass and convert value with `NSSet` to `Set`, then we can use all entity as usual inside SwiftUI structs

## [Day 57 - Project 12, part 1](https://www.hackingwithswift.com/100/swiftui/57)

More about Core Data

`\.self` works for anything that conforms to Hashable, because Swift will generate the hash value for the object and use that to uniquely identify it. This also works for Core Data’s objects because they already conform to Hashable

![](/assets/img/2020-08-23-week-1-1.png) If we choose `Manual/None`

![](/assets/img/2020-08-23-week-1-2.png) Then we can create our own `NSManagedObject` subclass to allow customization to the `NSManagedObject`

We can now remove `?` from each property, but it doesn't stop us from getting `nil` value, if we create computed properties when fetch values, then we can make sure we always have values just like below

```swift
public var wrappedTitle: String {
    title ?? "Unknown Title"
}
```

`hasChanges` can be used before trying to save managed object context to make sure there are something to save before saving

```swift
if self.moc.hasChanges {
    try? self.moc.save()
}
```

Also there is a way to prevent saving duplicates

![](/assets/img/2020-08-23-week-1-3.png) We can add constrains as shown above, add `import CoreData` in `SceneDelegate.swift`, add the following line to `willConnectTo` method. This will allow only unique object to be stored base on the constraints set.

```swift
context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
```

## [Day 56 - Project 11, part 4](https://www.hackingwithswift.com/100/swiftui/56)

After a few days of following along with me making a project, it’s time for you to step out of your comfort zone and start writing your own code. Once again, these are challenges I’m setting you based on everything you’ve learned so far, which means they are absolutely within your grasp if you set your mind to it.

Amy Morin, a social worker turned author, once said “the more you practice tolerating discomfort, the more confidence you'll gain in your ability to accept new challenges.” This is the underlying goal in all these challenges: giving you a little nudge to try something yourself, to figure out what works, and – bluntly – to screw up a few times before you figure out the right solution.

There is value in getting things right, but there’s just as much value in getting things wrong. Embrace that – learn to tolerate the discomfort that goes hand-in-hand with writing fresh code yourself – and you’ll be a great developer.

**Today you should work through the wrap up chapter for project 11, complete its review, then work through all three of its challenges.**

- [Bookworm: Wrap up](https://www.hackingwithswift.com/books/ios-swiftui/bookworm-wrap-up)
- [Review for Project 11: Bookworm](https://www.hackingwithswift.com/review/ios-swiftui/bookworm)

Share something online about what you learned, or how you might use it in the future – do you like Core Data? Are you keen to create more custom user interface components? Tell folks – stay accountable!

**Challenge**

One of the best ways to learn is to write your own code as often as possible, so here are three ways you should try extending this app to make sure you fully understand what’s going on.

1. Right now it’s possible to select no genre for books, which causes a problem for the detail view. Please fix this, either by forcing a default, validating the form, or showing a default picture for unknown genres – you can choose.
    - Code

        ```swift
        // Validate genre and set showNoGenreAlert to true if genre is empty
        func validateGenre() -> Bool {
            if self.genre.isEmpty {
                self.showNoGenreAlert.toggle()
                return false
            }
            return true
        }

        // Base alert on showNoGenreAlert value
        .alert(isPresented: $showNoGenreAlert) { () -> Alert in
            Alert(title: Text("Please select genre"), message: Text("Genre is required to save a book"), dismissButton: .cancel())
        }
        ```

    - Screenshot

        ![](/assets/img/2020-08-23-week-1-4.png)

2. Modify **`ContentView`** so that books rated as 1 star have their name shown in red.
    - Code

        ```swift
        // Choose color base on rating
        Text(book.title ?? "Unknown title").font(.headline).foregroundColor(book.rating < Int16(2) ? Color.red : Color.black)
        ```

    - Screenshot

        ![](/assets/img/2020-08-23-week-1-5.png)

3. Add a new “date” attribute to the Book entity, assigning **`Date()`** to it so it gets the current date and time, then format that nicely somewhere in **`DetailView`**.
    - Code

        ```swift
        // Add another date property to Bookworm.xcdatamodeld Book entity
        // Getting date from user using DatePicker
        DatePicker("Enter a date", selection: $date, displayedComponents: [.date, .hourAndMinute]).datePickerStyle(WheelDatePickerStyle())

        // When displaying, getting date string from date object 
        func dateString(from date: Date?) -> String {
            guard let date = date else { return "No Date available for the book" }
            let formatter = DateFormatter()
            formatter.timeStyle = .short
            formatter.dateStyle = .short
            return formatter.string(from: date)
        }
        ```

    - Screenshot

        ![](/assets/img/2020-08-23-week-1-6.png)