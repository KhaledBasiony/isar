<p align="center">
  <a href="https://isar.dev">
    <img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/isar.svg?sanitize=true" height="128">
  </a>
  <h1 align="center">Isar Database</h1>
</p>

<p align="center">
  <a href="https://pub.dev/packages/isar">
    <img src="https://img.shields.io/pub/v/isar?label=pub.dev&labelColor=333940&logo=dart">
  </a>
  <a href="https://github.com/isar/isar/actions/workflows/test.yml">
    <img src="https://img.shields.io/github/workflow/status/isar/isar/Dart%20CI/main?label=tests&labelColor=333940&logo=github">
  </a>
  <a href="https://app.codecov.io/gh/isar/isar">
    <img src="https://img.shields.io/codecov/c/github/isar/isar?logo=codecov&logoColor=fff&labelColor=333940">
  </a>
  <a href="https://t.me/isardb">
    <img src="https://img.shields.io/static/v1?label=join&message=isardb&labelColor=333940&logo=telegram&logoColor=white&color=229ED9">
  </a>
  <a href="https://twitter.com/simonleier">
    <img src="https://img.shields.io/twitter/follow/simonleier?style=flat&label=Follow&color=1DA1F2&labelColor=333940&logo=twitter&logoColor=fff">
  </a>
</p>

<p align="center">
  <a href="https://isar.dev">Quickstart</a> •
  <a href="https://isar.dev/schema">Documentation</a> •
  <a href="https://github.com/isar/samples">Sample Apps</a> •
  <a href="https://github.com/isar/isar/discussions">Support & Ideas</a> •
  <a href="https://pub.dev/packages/isar">Pub.dev</a>
</p>

> #### Isar [ee-zahr]:
>
> 1. River in Bavaria, Germany.
> 2. [Crazy fast](#benchmarks) NoSQL database that is a joy to use.

## Features

- 💙 **Made for Flutter**. Easy to use, no config, no boilerplate
- 🚀 **Highly scalable** The sky is the limit (pun intended)
- 🍭 **Feature rich**. Composite & multi-entry indexes, query modifiers, JSON support etc.
- ⏱ **Asynchronous**. Parallel query operations & multi-isolate support by default
- 🦄 **Open source**. Everything is open source and free forever!

Isar database can do much more (and we are just getting started)

- 🕵️ **Full-text search**. Make searching fast and fun
- 📱 **Multiplatform**. iOS, Android, Desktop and FULL WEB SUPPORT!
- 🧪 **ACID semantics**. Rely on database consistency
- 💃 **Static typing**. Compile-time checked and autocompleted queries
- ✨ **Beautiful documentation**. Readable, easy to understand and ever-improving

Join the [Telegram group](https://t.me/isardb) for discussion and sneak peaks of new versions of the db.

If you want to say thank you, star us on GitHub and like us on pub.dev 🙌💙

## Quickstart

Holy smokes you're here! Let's get started on using the coolest Flutter database out there...

### 1. Add to pubspec.yaml

```yaml
dependencies:
  isar: 3.0.0-dev.13
  isar_flutter_libs: 3.0.0-dev.13 # contains Isar Core

dev_dependencies:
  isar_generator: 3.0.0-dev.13
  build_runner: any
```

or through the command line using flutter (or dart):
```console
flutter pub add isar:3.0.0-dev.13 isar_flutter_libs:3.0.0-dev.13
flutter pub add --dev isar_generator:3.0.0-dev.13 build_runner
flutter pub get
```

### 2. Annotate a Collection

inside the `lib` directory (or a directory inside it), add this to an `email.dart` file
```dart
import 'package:isar/isar.dart';

part 'email.g.dart';

@collection
class Email {
  Id id = Isar.autoIncrement; // you can also use id = null to auto increment

  String? title;

  List<Recipient>? recipients;

  @enumerated
  Status status = Status.pending;
}

@embedded
class Recipient {
  String? name;

  String? address;
}

enum Status {
  draft,
  sending,
  sent,
}
```
then run the following command:  
```console
flutter pub run build_runner build
```

### 3. Open a database instance

```dart
final isar = await Isar.open([EmailSchema]);
```

### 4. Query the database

```dart
final emails = await isar.emails.filter()
  .titleContains('awesome', caseSensitive: false)
  .sortByDateDesc()
  .limit(10)
  .findAll();
```

## Isar Database Inspector

The [Isar Inspector](https://github.com/isar/isar/releases/latest) allows you to inspect the Isar instances & collections of your app in real-time. You can execute queries, switch between instances and sort the data.

<img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/isar-inspector.png?sanitize=true">

## CRUD operations

All basic crud operations are available via the `IsarCollection`.

```dart
final newPost = Post()..title = 'Amazing new database';

await isar.writeTxn(() async {
  newPost.id = await isar.posts.put(newPost); // insert & update
});

final existingPost = await isar.posts.get(newPost.id!); // get

await isar.writeTxn(() async {
  await isar.posts.delete(existingPost.id!); // delete
});
```

## Database Queries

Isar database has a powerful query language that allows you to make use of your indexes, filter distinct objects, use complex `and()`, `or()` and `.xor()` groups, query links and sort the results.

```dart
final usersWithPrefix = isar.users
  .where()
  .nameStartsWith('dan') // use index
  .limit(10)
  .findAll()

final usersLivingInMunich = isar.users
  .filter()
  .ageGreaterThan(32)
  .or()
  .addressMatches('*Munich*', caseSensitive: false) // address containing 'munich' (case insensitive)
  .optional(
    shouldSort, // only apply if shouldSort == true
    (q) => q.sortedByAge(),
  )
  .findAll()
```

## Links

You can easily define relationships between objects. In Isar database they are called links and backlinks:

```dart
@collection
class Teacher {
  Id? id;

  late String subject;

  @Backlink(to: 'teacher')
  final students = IsarLinks<Student>();
}

@collection
class Student {
  Id? id;

  late String name;

  final teacher = IsarLink<Teacher>();
}
```

## Database Watchers

With Isar database, you can watch collections, objects, or queries. A watcher is notified after a transaction commits successfully and the target actually changes.
Watchers can be lazy and not reload the data or they can be non-lazy and fetch new results in the background.

```dart
Stream<void> collectionStream = isar.posts.watchLazy;

Stream<List<Post>> queryStream = databasePosts.watch();

queryStream.listen((newResult) {
  // do UI updates
})
```

## Isar Database Benchmarks

Benchmarks only give a rough idea of the performance of a database but as you can see, Isar NoSQL database is quite fast 😇

<img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/benchmarks/insert.png" width="100%" /> | <img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/benchmarks/query.png" width="100%" />
--- | ---
<img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/benchmarks/delete.png" width="100%" /> | <img src="https://raw.githubusercontent.com/isar/isar/main/.github/assets/benchmarks/size.png" width="100%" />

If you are interested in more benchmarks or want to check how Isar performs on your device you can run the [benchmarks](https://github.com/isar/isar_benchmark) yourself.

## Unit tests

If you want to use Isar database in unit tests or Dart code, call `await Isar.initializeIsarCore(download: true)` before using Isar in your tests.

Isar NoSQL database will automatically download the correct binary for your platform. You can also pass a `libraries` map to adjust the download location for each platform.

Make sure to use `flutter test -j 1` to avoid tests running in parallel. This would break the automatic download.

## Contributors ✨

Thanks go to these wonderful people:

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/Frostedfox"><img src="https://avatars.githubusercontent.com/u/84601232?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Frostedfox</b></sub></a><br /><a href="https://github.com/isar/isar/commits?author=Frostedfox" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/h1376h"><img src="https://avatars.githubusercontent.com/u/3498335?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Hamed H.</b></sub></a><br /><a href="https://github.com/isar/isar/commits?author=h1376h" title="Code">💻</a> <a href="#maintenance-h1376h" title="Maintenance">🚧</a></td>
    <td align="center"><a href="https://github.com/Jtplouffe"><img src="https://avatars.githubusercontent.com/u/32107801?v=4?s=100" width="100px;" alt=""/><br /><sub><b>JT</b></sub></a><br /><a href="https://github.com/isar/isar/commits?author=Jtplouffe" title="Tests">⚠️</a> <a href="https://github.com/isar/isar/issues?q=author%3AJtplouffe" title="Bug reports">🐛</a></td>
    <td align="center"><a href="http://achim.io"><img src="https://avatars.githubusercontent.com/u/43643339?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Joachim Nohl</b></sub></a><br /><a href="#maintenance-nohli" title="Maintenance">🚧</a></td>
    <td align="center"><a href="https://github.com/Moseco"><img src="https://avatars.githubusercontent.com/u/10720298?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Moseco</b></sub></a><br /><a href="https://github.com/isar/isar/issues?q=author%3AMoseco" title="Bug reports">🐛</a></td>
    <td align="center"><a href="https://github.com/Viper-Bit"><img src="https://avatars.githubusercontent.com/u/24822764?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Peyman</b></sub></a><br /><a href="https://github.com/isar/isar/issues?q=author%3AViper-Bit" title="Bug reports">🐛</a> <a href="https://github.com/isar/isar/commits?author=Viper-Bit" title="Code">💻</a></td>
    <td align="center"><a href="https://www.linkedin.com/in/simon-leier/"><img src="https://avatars.githubusercontent.com/u/13610195?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Simon Leier</b></sub></a><br /><a href="https://github.com/isar/isar/issues?q=author%3Aleisim" title="Bug reports">🐛</a> <a href="https://github.com/isar/isar/commits?author=leisim" title="Code">💻</a> <a href="https://github.com/isar/isar/commits?author=leisim" title="Documentation">📖</a> <a href="https://github.com/isar/isar/commits?author=leisim" title="Tests">⚠️</a> <a href="#example-leisim" title="Examples">💡</a></td>
    <td align="center"><a href="https://github.com/blendthink"><img src="https://avatars.githubusercontent.com/u/32213113?v=4?s=100" width="100px;" alt=""/><br /><sub><b>blendthink</b></sub></a><br /><a href="#maintenance-blendthink" title="Maintenance">🚧</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This database project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind are welcome!

### License

```
Copyright 2022 Simon Leier

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
