***compatible version***
Using package
	-auto_route: 7.8.4 `flutter pub add auto_route:7.8.4`
	-auto_route_generator: 7.3.1 `flutter pub add dev:auto_route_generator:7.3.1`
	-build_runner `dart pub add dev:build_runner`
	
สร้างไฟล์ routers
```dart
import 'package:auto_route/auto_route.dart';

part 'routers.gr.dart';

@AutoRouterConfig()
class AppRouter extends _$AppRouter {
	@override
	List<AutoRoute> get routes => [];
}
```

ใช้คำสั่ง generate file `dart run build_runner build` จะได้ไฟล์ `routers.gr.dart`

Config route ที่ main.dart
```dart
import 'package:flutter/material.dart';
import 'package:flutter_poc_package/core/navigations/routers.dart';

void main() {
	runApp(MyApp());
}

class MyApp extends StatelessWidget {
	MyApp({super.key});
	
	final _appRouter = AppRouter();
	
	@override
	Widget build(BuildContext context) {
		return MaterialApp.router(
			routerConfig: _appRouter.config(),
		);
	}
}
```

ต่อไปเป็นการเพิ่ม Route (สร้าง page)
```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

@RoutePage()
class HomeScreen extends StatelessWidget {
	const HomeScreen({super.key});
	
	@override
	Widget build(BuildContext context) {
		return const Scaffold(
			body: Center(
			child: Text('Home Screen'),
			),
		);
	}
}
```
แล้วสั่ง `dart run build_runner build` ใหม่ และเพิ่ม page เข้าไป
```dart
import 'package:auto_route/auto_route.dart';
import 'routers.gr.dart';

@AutoRouterConfig()
class AppRouter extends $AppRouter {
	@override
	List<AutoRoute> get routes => [AutoRoute(page: HomeRoute.page)];
}
```