---
theme: bricks # the-unnamed
title: "Riverpod 3.0: spoiler alert!"
class: text-right
highlighter: shiki
drawings:
  persist: false
transition: fade
mdc: true
---

# Riverpod 3.0
Spoiler alert! 🎯🔥


---
title: Chi sono?
layout: image-left
image: https://riverpod.dev/img/logo.png
backgroundSize: 70%
---

# Ciao!
<v-clicks>

**Chi sono?**
 
- 🎯 Dart enjoyer
- 🏞️ Riverpod supporter
- ⚗️ Functional programming!

</v-clicks>

<v-clicks>

**Che si fa oggi?**

- 📚 Refresher: cos'è Riverpod?
- 🔥 Riverpod 3.0?
- 🧪 Q/A

*Attenzione: NO CODE PERMITTED!* 🚫

</v-clicks>


<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/lucavenir" target="_blank" alt="GitHub" title="~venir on GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
    <a href="https://twitter.com/venir_dev" target="_blank" alt="X" title="~venir on X"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-x />
  </a>
</div>

<!--
all'inizio: cose noiose, breaking change, dettagli...

e poi, verso la fine, catturerò la vostra attenzione con fresche novità
-->

---
---

# Business logic, with Riverpod

**Providers**: *functions with benefits*.
<br/>

<v-click>
Da ...
</v-click>
```dart {1|2|all}
Future<int> check() {
  return Future.delayed(200.milliseconds, () => ...);
}
```
<br/>
<v-click>
... a:
</v-click>

```dart {none|1|2|3|all}
@riverpod
Future<int> check(CheckRef ref) {
  return Future.delayed(200.milliseconds, () => ...);
}
```

---
---

# Business logic, with Riverpod
```dart {1-6|8-10|all}
@riverpod
class CheckController extends _$CheckController {
  @override
  Future<int> build() {
    return Future.delayed(200.milliseconds, () => 0);
  }

  Future<void> moreSpritz() {
    return update((state) => Future.delayed(200.milliseconds, () => state + 1));
  }
}
```

<br/>

```dart {none|1-2|3|4|all}
@riverpod
Future<int> fraudCheck(FraudCheckRef ref) async {
  final check = await ref.watch(checkProvider.future);
  return 2 * check;
}
```


---
---

# Final result

```dart {none|1-2|3|8-13|14-17|all}
class CheckPleaseWidget extends ConsumerWidget {
  Widget build(BuildContext context, WidgetRef ref) {
    final AsyncValue<int> fraud = ref.watch(fraudCheckProvider);
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      mainAxisSize: MainAxisSize.min,
      children: [
        switch (fraud) {
          AsyncData(value: 0) => const Text('Cosa ti porto? 👀'),
          AsyncData(:final value) => Text('Sono € $value, grazie 😸'),
          AsyncError() => const Text('Mi spiace, niente spriz oggi 😢'),
          _ => const Image.network('draft-wine-gif-url')
        },
        ElevatedButton(
          onPressed: ref.read(checkControllerProvider.notifier).moreSpritz,
          child: const Text('🍹'),
        ),
      ],
    );
  }
}
```

<div class="abs-tr m-6 flex gap-2">
  <a href="https://media.tenor.com/q_Pzz7xscmwAAAAM/glass-champagne-pouring-bubbley.gif" target="_blank" alt="Spritz" title="Draft a Spritz!"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    🍾
  </a>
</div>

---
---

## Spoiler #0: `sealed class AsyncValue<T>`

<v-clicks>

💈 `AsyncValue` ora è `sealed`

⚠️ `AsyncData`/`AsyncLoading`/`AsyncError` ora sono `final`

```dart {none|all|9|5-10}
return Column(
  mainAxisAlignment: MainAxisAlignment.center,
  mainAxisSize: MainAxisSize.min,
  children: [
    switch (fraud) {
      AsyncData(value: 0) => const Text('Cosa ti porto? 👀'),
      AsyncData(:final value) => Text('Sono € $value, grazie 😸'),
      AsyncError() => const Text('Mi spiace, niente spriz oggi 😢'),
      AsyncLoading() => const Image.network('draft-wine-gif-url')
    },
    ElevatedButton(
      onPressed: ref.read(checkControllerProvider.notifier).moreSpritz,
      child: const Text('🍹'),
    ),
  ],
);
```

</v-clicks>

<!-- tecnicamente questa è una breaking change... -->

---
---

## Spoiler #1: API pulite e semplificate

<v-clicks>

🗑️ cleanup delle *public api* (~90 tipi rimossi)

⚡ performance improvements

🐛 fixati 50 bug

</v-clicks>


<v-clicks depth="2">

- "Addio" StateNotifier/StateProvider/... 
  - ➡️ Verrà spostato in un package dedicato: `riverpod/legacy.dart`
- Addio `AutoDispose`: ora *tutti* i provider sono `AutoDispose`, non servirà specificarlo più!
  - 👋 `AutoDisposeNotifier`, `AutoDisposeProvider`, etc.
  - Vi serve un `AlwaysAlive` provider / notifier? usate `ref.keepAlive`
- Addio a tante (strane?) sottoclassi di `Ref`
  - 👋 `ProviderRef`, `StreamProviderRef`; ma anche `AutoDisposeFutureProviderRef`, etc.
  - ... Aspetta, cosa? Perché? 🤔

</v-clicks>


<!--
Ma come, *davvero* usate `AlwaysAliveProviderBase`? Non vi crede nessuno...

Momento di discussione. Per tre motivi:
 (1) ora è possibile scrivere estensioni e mixin per provider e notifier
 (2) codebase ridotta del 50%
 (3) abilita una nuova e richiestissima feature...
-->

---
---
## Spoiler #2: Generics

<v-click>

Sarà possibile definire provider basati su un tipo `T` generico.


```dart {none|1-3|4-5|6-8|all}
// warning: pseudo code!!
@riverpod
class MyListNotifier<T> extends _$MyListNotifier<T> {
  @override
  List<T> build() => [];

  void add(T value) {
    state = [...state, value];
  }
}
```
</v-click>


<v-click>

➡️ in arrivo istruzioni per l'uso..!

</v-click>


---
---

## Spoiler #3: `if (mounted) ...`


<v-clicks>

🚨 Breaking change!

Riverpod `2.0`: tra un `build` e l'altro `Notifier` non veniva `dispose`.

Riverpod `3.0`: `Notifier` segue lo stesso ciclo di vita di `build`.


```dart
@riverpod
class SomeNotifier extends _$SomeNotifier {
  @override
  int build() {
    Future.delayed<void>(3.seconds, ref.invalidateSelf);
    // Riverpod 2: tra tre secondi `build` viene re-eseguito
    // Riverpod 3: l'intera classe `SomeNotifier` viene `dispose` e re-allocata, `build` incluso
    return 0;
  }
}
```

</v-clicks>

---
---
## Spoiler #3: `if (mounted) ...`


Perché questo cambiamento? 🤔

<v-clicks>

```dart {all|6-11|13}
@riverpod
class SomeNotifier extends _$SomeNotifier {
  @override
  int build() => ...;
  
  Future<void> asyncMethod() async {
    await something();

    if (!mounted) return;
    ref.something();
  }

  var _internalState = 0; // look out!
}
```

</v-clicks>




---
---

## Spoiler #4: Testing utilities

<v-clicks>

👀 Tutti noi testiamo il nostro codice, vero?

🧪 E infatti, abbiamo bisogno di test utilities!

```dart
// Riverpod 2.0
ProviderContainer testContainer({List<ProviderOverride>? overrides}) {
  final container = ProviderContainer(overrides: []);
  addTearDown(container.dispose);
  
  return container;
}

test('my test', () {
  final container = testContainer();
  // TODO use container to test your code here 😏
})
```

</v-clicks>

---
---

## Spoiler #4: Testing utilities

<v-clicks>

Riverpod 3.0: è in arrivo... `ProviderContainer.test` 😜

```dart
// Riverpod 3.0
test('my test', () {
  // warn: pseudocode
  final container = ProviderContainer.test();
  // TODO use container to test your code here 😏
})
```

</v-clicks>

---
---

## Spoiler #5: Lazy `ref.listen`

<v-clicks>

```dart
@riverpod
int printMe(PrintMeRef ref) {
  print('me!');
  return 0;
}
```

```dart
@riverpod
int another(AnotherRef ref) {
  // warn: pseudocode!!
  ref.listen(printMeProvider, weak: true, (prev, next) {});

  return 42;
}
```

😴 La stringa `me!` non viene stampata.

</v-clicks>

---
---

## Spoiler #6: Side effects, ma meglio!

<v-clicks>

📖 Sicuramente avete letto [la documentazione](https://riverpod.dev/docs/essentials/side_effects#going-further-showing-a-spinner--error-handling)... vero?

```dart
class MyAsyncNotifier extends _$MyAsyncNotifier {
  @override
  Future<int> build() => ...;
  
  // Riverpod 2.0: è responsabilità tua gestire la UX per questo effetto collaterale!
  Future<void> sideEffect() async {
    await something();
  }
}
```
 
</v-clicks>

---
---

## Spoiler #6: Side effects, ma meglio!

<v-clicks>

➡️ In arrivo... query mutations!

```dart
class MyAsyncNotifier extends _$MyAsyncNotifier {
  @override
  Future<int> build() => ...;
  
  @mutation  // warn: this is pseudocode.
  Future<void> sideEffect() async {
    await something();
  }
}
```

</v-clicks>

---
---

## Spoiler #6: Side effects, ma meglio!

<v-clicks>

```dart{all|1-2|3-6|7-10|11-18|19-22|all}
// warn: *tons* of pseudocode
... child: switch (ref.watch(sideEffect)) { 
  Empty() => ElevatedButton(
    onPressed: () => sideEffect(),
    child: Text('Send 📧'),
  ),
  Loading() => ElevatedButton(
    onPressed: null,
    child: CircularProgressIndicator(),
  ),
  Errored() => ElevatedButton(
    onPressed: query.retry,
    style: ButtonStyle(
      backgroundColor: WidgetStateProperty.all(Colors.redAccent),
      foregroundColor: WidgetStateProperty.all(Colors.white),
    ),
    child: Text('Oof. Try again?'),
  ),
  Success() => ElevatedButton(
    onPressed: null,
    child: Icon(Icons.check),
  ),
};
```

</v-clicks>

---
---

## Spoiler #7: Retry w/ exponential backoff

<v-clicks>

```dart
@riverpod
Future<int> unreliable(UnreliableRef ref) async {
  final someResult = await someRepo.fromNetwork(throws: true);  // fails 100% of the time
  return someResult;
}
```

```dart
Duration? myRetry(int retryCount, Object error) {
  // your logic here, e.g.
  if (retryCount > 10) return null;

  final retryDelay = 1 + retryCount * retryCount;
  return retryDelay.seconds;
}
```

</v-clicks>

---
---

## Spoiler #7: Retry w/ exponential backoff

<v-clicks>

```dart
// warn: pure imaginative pseudocode
@Riverpod(retry: myRetry)
Future<int> unreliable(UnreliableRef ref) async {
  final someResult = await someRepo.fromNetwork(throws: true);  // fails 100% of the time
  return someResult;  // tries again after 1 sec, 2 secs, 5 secs, 10 secs...
}
```

⚠️ (probably) more assembly required

</v-clicks>

---
---

## Spoiler #8: Offline caching (!!)

<v-clicks>

Reminder: what is riverpod?

> A Reactive Caching and Data-binding Framework

What if...

```dart
// warn: 110% imaginative example
ProviderScope(
  // TODO: define your offline connector / adapter here (based on your database preference)
  offlineConnector: const SharedAppPreferenceAsJson(),  // example
)
```

</v-clicks>

---
---

## Spoiler #8: Offline caching (!!)

<v-clicks>

Then... 
```dart
// warn: this is my own pure imagination, not even pseudocode at this point
@Riverpod(offline: 'tableName', retry: myRetry)
Future<List<Product>> products(ProductsRef ref) {
  final response = http.get('my-api/products');
  
  return (response as List).map(Product.fromJson).toList();
}
```

⚠️ (certainly) *way* more assembly required!

</v-clicks>

<!--

L'offline richiede tantissima "cura" e logica "in più" che andrà definita "da qualche parte"
Ad oggi non è chiaro dove.

Ad esempio, per gestire un db locale bisogna pensare a:
  - definizione e migrazioni
  - logica di scelta tra db e api (dove leggiamo i dati?)
  - logica di sincronizzazione tra RAM e db: quand'è che aggiorniamo il db?
  - logica di invalidazione della cache db: coincide con la cache in RAM?
-->

---
---

## Wrap up


<v-clicks>

☣️ Alcune slide sono *volutamente* "imprecise"

📖 Serve tantissima documentazione...

🩻 C'è ancora molto altro...!!

🎯 Rilascio 3.0 *con* documentazione

🌠 Preview incoming soon-ish...

🆕 Stable release later in 2024/25..?

💭 Q/A... oppure chiacchieriamo

</v-clicks>

---
layout: center
---

# Grazie per l'attenzione
