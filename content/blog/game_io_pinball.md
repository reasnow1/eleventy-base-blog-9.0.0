---
title: 谷歌I/O弹球适配 Removing Firebase from I/O Pinball A Clean Slate
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-24
tags: 游戏开发Games
---
此项目耗费巨大，最后成果，由于他这个Flutter实在吃配置，低配手机几乎无法发射球，屏幕小的手机连右下角火箭都看不到，因此我不打算继续研究了，此篇博客作存档。

下面是Deepseek写的，我作一些补充。

Here's a blog post in Markdown format you can publish. It tells the story of removing Firebase from the I/O Pinball game, including the technical steps, challenges, and final result.

# Removing Firebase from I/O Pinball: A Clean Slate

I recently downloaded the official I/O Pinball game from GitHub – a beautiful Flutter pinball machine built for Google I/O 2022. The game works great, but it requires a Firebase project for anonymous authentication and Firestore to store high scores.

下载地址：
[https://github.com/flutter/pinball](https://github.com/flutter/pinball)

他的源码需要Firebase所以我们要移除这个Firebase。

I didn’t want to set up Firebase. I just wanted to play pinball offline, without any cloud dependencies. So I decided to **remove Firebase entirely** from the source code.

Here’s how I did it, the problems I ran into, and the final result – a fully playable, standalone pinball game.

## Why Remove Firebase?

The README clearly states that Firebase is used only for:
- Anonymous authentication
- Firestore (high score leaderboard)

That’s it. No analytics, no remote config. So removing Firebase would not break the core pinball mechanics – just the online scoreboard.

工具准备：页面上有标注一个Flutter 3.22 (#505)的修改，所以，我们使用3.22版本。

官网参考：
[https://docs.flutter.dev/install/](https://docs.flutter.dev/install/)
```bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install -y curl git unzip xz-utils zip libglu1-mesa
git clone https://github.com/flutter/flutter.git
cd flutter
git checkout 3.22.0
```
我是用了VScode来安装，然后，版本不对，后面自己去里面改版本了。

加入到PATH：
```bash
sudo nano ~/.bashrc
```
最后一行加
```config
export PATH="$PATH:$HOME/flutter/bin"
```
```bash
source ~/.bashrc
```
```bash
flutter doctor
```
检验安装。

## Step 1: Identify All Firebase Dependencies

First, I scanned the project structure:

```
lib/
├── firebase_options.dart
├── main.dart
├── app/view/app.dart
├── game/
│   ├── pinball_game.dart
│   └── components/backbox/
├── pubspec.yaml
```

The main offenders:
- `firebase_core`, `firebase_auth`, `cloud_firestore`
- `authentication_repository` and `leaderboard_repository` (internal packages)
- `lib/firebase_options.dart`


## Step 2: Remove Dependencies from `pubspec.yaml`

此时可以直接输入
```bash
flutter run -d chrome
```
来尝尝咸淡，然后他很快会告诉你依赖不对，是要3.4.0，他里面所有的都是3.4.3

这些行删除I deleted these lines from `pubspec.yaml`:

```yaml
firebase_core: ^x.x.x
firebase_auth: ^x.x.x
cloud_firestore: ^x.x.x
authentication_repository:
  path: packages/authentication_repository
leaderboard_repository:
  path: packages/leaderboard_repository
```
注意：
```yaml
environment:
  sdk: "^3.4.3"
```
我是直接笨方法，直接改3.4.0。然后，packages文件夹里有很多小文件夹，每个里面都有pubspec.yaml，也是硬着头皮全改完了。

Then ran `flutter pub get`.

上面这个运不运行都行，不运行，run的时候也会下载。

## Step 3: Strip Firebase from `main.dart`

Original code had `Firebase.initializeApp`, `AuthenticationRepository`, and anonymous sign‑in. I replaced everything with simple stub classes that do nothing.

But that led to type mismatches – the `App` widget expected specific repository types. So I decided to **completely remove** the authentication and leaderboard parameters from the `App` constructor.

按他说的改。

## Step 4: Modify `app.dart`

I removed the two parameters and their providers:

```dart
class App extends StatelessWidget {
  const App({
    Key? key,
    required ShareRepository shareRepository,
    required PinballAudioPlayer pinballAudioPlayer,
    required PlatformHelper platformHelper,
  }) : ...
```

No more `AuthenticationRepository` or `LeaderboardRepository`.

## Step 5: Clean Up `pinball_game.dart`

The `PinballGame` class had a `leaderboardRepository` field, a `preFetchLeaderboard` method, and passed `leaderboardRepository` to the `Backbox` component.

I deleted:
- The field and constructor parameter
- The `_entries` list and `preFetchLeaderboard` method
- The `FlameProvider<LeaderboardRepository>` from the provider list
- The arguments to `Backbox` constructor

## Step 6: Simplify `Backbox`

The `Backbox` was the most complex part – it had a whole BLoC for leaderboard states (loading, success, failure, initials form, share). I replaced the entire file with a simple visual component:

```dart
class Backbox extends PositionComponent {
  Backbox();

  @override
  Future<void> onLoad() async {
    // just add the background sprite
  }
}
```

But wait – the game called `requestInitials` on the `Backbox` when the game ended. Without that method, the game would soft‑lock at game over.

## Step 7: Fix the Game Over Soft‑Lock

Originally, `game_bloc_status_listener.dart` would call:

```dart
gameRef.descendants().whereType<Backbox>().first.requestInitials(...)
```

This opened the initials input form (to save high score). After removing Firebase, that call did nothing, and the replay button never appeared.

My solution: **re‑implement `requestInitials` to simply show the replay button overlay** instead of opening a form.

```dart
void requestInitials({...}) {
  gameRef.overlays.add(PinballGame.replayButtonOverlay);
}
```

Now when the game ends, the replay button appears immediately. No Firebase, no leaderboard, no soft‑lock.

## Step 8: Remove Unused Files

I deleted:
- `lib/game/components/backbox/bloc/` (entire directory)
- `lib/game/components/backbox/displays/` (all leaderboard UI widgets)
- `lib/firebase_options.dart`
- `firestore.rules` and `storage.rules`

## The Result

先不急，加一些中文：
在

pinball-main/lib/l10n/arb/app_en.arb

这里全是英文，按官方的可以做一个中文，我是直接中英混合。

After all changes, the game compiles and runs perfectly:

```bash
flutter run -d chrome
```

- The pinball mechanics are untouched
- No network calls or Firebase errors
- Game over shows a replay button
- The backbox is now just a beautiful decoration

## What Was Lost (and Why It’s Okay)

- **Online high scores** – gone. But for a local offline game, that’s fine.
- **Anonymous authentication** – removed. No need.
- **Sharing scores** – also removed. Could be re‑added later with a simple share dialog.

## What Remains

- Full pinball gameplay (flippers, bumpers, multiball, bonuses)
- Audio and voice‑over
- Character selection
- Local game state management

## Lessons Learned

1. **Firebase is easy to remove** when used only for non‑critical features.
2. **Stub methods** can keep the game flow intact without rewriting large parts.
3. **Always test the game over sequence** – that’s where hidden dependencies often surface.
4. **A little patience** – the first few compilations will throw many errors. Fix them one by one.

## Try It Yourself

You can download the original game from [flutter/pinball](https://github.com/flutter/pinball) and apply these changes. The full modified source code (without Firebase) is available on my GitHub (link).

Now go play some pinball – no internet required 🚀

---

*Enjoyed this post? Let me know on Twitter or leave a comment below.*


You can adjust the author name, date, and add your own GitHub link. This post should be ready to publish on your blog. Congratulations on successfully removing Firebase!

## 建造
测试好后，建造三连击
```bash
flutter clean
flutter pub get
flutter build web
```
此时还不能运行，看控制台：
```console
Uncaught SyntaxError: missing ) after argument list
index.html:121:57
```
记事本打开后找到
```js
} else if (!reg.active.scriptURL.endsWith(""2785492232"")) {
```
双重引号，改成一个引号。这数字，改过内容后，数字还不一样。
```js
} else if (!reg.active.scriptURL.endsWith("2785492232")) {
```

后面的script有谷歌的分析，可以全删了：
```html
  <script>
    (function (w, d, s, l, i) {
      w[l] = w[l] || []; w[l].push({
        'gtm.start':
          new Date().getTime(), event: 'gtm.js'
      }); var f = d.getElementsByTagName(s)[0],
        j = d.createElement(s), dl = l != 'dataLayer' ? '&l=' + l : ''; j.async = true; j.src =
          'https://www.googletagmanager.com/gtm.js?id=' + i + dl; f.parentNode.insertBefore(j, f);
    })(window, document, 'script', 'dataLayer', 'GTM-ND4LWWZ');
  </script>
  <script>
    (function (i, s, o, g, r, a, m) {
      i['GoogleAnalyticsObject'] = r; i[r] = i[r] || function () {
        (i[r].q = i[r].q || []).push(arguments)
      }, i[r].l = 1 * new Date(); a = s.createElement(o),
        m = s.getElementsByTagName(o)[0]; a.async = 1; a.src = g; m.parentNode.insertBefore(a, m)
    })(window, document, 'script', '//www.google-analytics.com/analytics.js', 'ga');
    ga('create', 'UA-67589403-1', 'auto');
    ga('send', 'pageview');
  </script>
```

此时还没完，音频会无法加载，音频是从根目录读取。修复操作：
```bash
pinball-main/build/web$ ls
__         favicon.png           flutter_service_worker.js  main.dart.js
assets     flutter_bootstrap.js  icons                      manifest.json
canvaskit  flutter.js            index.html                 version.json
pinball-main/build/web$ mkdir packages
pinball-main/build/web$ cp -r ./assets/packages/pinball_audio ./packages/
```
外部依赖替换：改main.dart.js，关键点：
```js
https://www.gstatic.com/flutter-canvaskit
https://fonts.gstatic.com/s/
```
之后可以
```
python3 -m http.server
```
试试了。

放上去的时候，由于他这些玩意都是读根目录，愣是要我单独开了一个端口给它运行。

