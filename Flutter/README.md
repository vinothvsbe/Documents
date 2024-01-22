# Flutter

Command to create new flutter project
```bash
flutter create --org com.grayprogrammer mycalculator
```
> They are called reverse domain identifiers

#### Adding new package to flutter
Following command will help to add flutter package to flutter
```bash
flutter pub add flutter_staggered_grid_view
```
If you go and check pubspec.yaml then we can see that `flutter_staggered_grid_view`

> Point to Remeber : Pub add will always add the latest version available.

To view all the packages we have to use http://pub.dev/packages/flutter_staggered_grid_view

### List of useful plugins
| Plugin Name | Plugin Description |
------------------------------------
| Plugin Name   | Plugin Description    | 
| ----------- | ----------- |
| flutter_staggered_grid_view | Use to create grid layout in flutter |
| equatable | We can use this package to compare anything equal, even if we want to compare two objects as well |


#### Factory class
To form any object from json data, following factory class can be used in flutter

```dart
factory Category.fromJson (
Map<String, dynamic>
json, {
String? id,
｝）｛
return Category (
id: id ?? jsonl'id']?? "',
name: json I'name'] ?? "',
description: json['description'l ??"', imageUrl:
json l'imageUrl']?? 'https://source.unsplash.com/random/?fashion'
｝
```
Code explanation:
This is a Dart factory constructor for a class named Category. Factory constructors are a way to implement a constructor for a class that doesn't always create a new instance of its class. Instead, it can return an existing instance or some other value. In this case, the factory constructor is used to create a new Category instance from a JSON object.

Here's a breakdown of the code:

Factory Constructor Definition:
**factory Category.fromJson(Map<String, dynamic> json, {String? id}):** This is the declaration of the factory constructor. It takes two parameters:
**json: A Map<String, dynamic>** representing the JSON object from which the Category instance will be created.
**id:** An optional parameter of type String?. The question mark indicates that id can be null.
Constructor Body:
The body of the constructor uses the return statement to create and return a new Category instance.
**id: id ?? json['id'] ?? '':** This sets the id of the Category instance. The expression uses the null-aware operator ??. It first checks if id is not null. If id is null, it then looks for an id in the json map. If it doesn't find an id in the JSON, it defaults to an empty string ''.
**name: json['name'] ?? '':** This sets the name property of the Category. It attempts to retrieve name from the json map. If name is not found, it defaults to an empty string ''.
**description: json['description'] ?? '':** Similar to name, this line sets the description property, defaulting to an empty string if description is not found in the JSON.
**imageUrl: json['imageUrl'] ?? 'https://source.unsplash.com/random/?fashion':** This sets the imageUrl property. If imageUrl is not in the JSON, it defaults to a random fashion image URL from Unsplash.


