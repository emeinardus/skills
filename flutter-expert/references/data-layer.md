# Data Layer Reference

## Clean Architecture — Data Flow

```
Remote API (JSON)
  → DTO (Data Transfer Object)         [data/dtos/]
  → Mapping via .toEntity()
  → Entity (pure Dart, no framework)   [domain/entities/]
  → Repository Interface               [domain/repositories/i_*.dart]
  → Repository Implementation          [data/repositories/]
  → Use Case                           [domain/use_cases/]
  → Notifier (Riverpod)                [application/]
  → Widget                             [presentation/]
```

**Never** let DTOs, Dio responses, or raw JSON reach the application or
presentation layer. Always map at the repository boundary.

---

## DTO Pattern

```dart
// features/user/data/dtos/user_dto.dart
class UserDto {
  const UserDto({
    required this.id,
    required this.firstName,
    required this.lastName,
    required this.email,
    required this.avatarUrl,
    required this.createdAt,
  });

  final String id;
  final String firstName;
  final String lastName;
  final String email;
  final String? avatarUrl;
  final String createdAt;

  factory UserDto.fromJson(Map<String, dynamic> json) {
    return UserDto(
      id: json['id'] as String,
      firstName: json['first_name'] as String,
      lastName: json['last_name'] as String,
      email: json['email'] as String,
      avatarUrl: json['avatar_url'] as String?,
      createdAt: json['created_at'] as String,
    );
  }

  Map<String, dynamic> toJson() => {
        'id': id,
        'first_name': firstName,
        'last_name': lastName,
        'email': email,
        'avatar_url': avatarUrl,
        'created_at': createdAt,
      };
}
```

---

## Entity Pattern

```dart
// features/user/domain/entities/user.dart
class User {
  const User({
    required this.id,
    required this.firstName,
    required this.lastName,
    required this.email,
    this.avatarUrl,
    required this.createdAt,
  });

  final String id;
  final String firstName;
  final String lastName;
  final String email;
  final String? avatarUrl;
  final DateTime createdAt;

  String get fullName => '$firstName $lastName';
}
```

---

## DTO → Entity Mapping

Mapping lives in an extension on the DTO — keeps entities clean:

```dart
// features/user/data/dtos/user_dto.dart (add extension at bottom)
extension UserDtoMapper on UserDto {
  User toEntity() {
    return User(
      id: id,
      firstName: firstName,
      lastName: lastName,
      email: email,
      avatarUrl: avatarUrl,
      createdAt: DateTime.parse(createdAt),
    );
  }
}

extension UserDtoListMapper on List<UserDto> {
  List<User> toEntities() => map((dto) => dto.toEntity()).toList();
}
```

---

## Repository Interface

```dart
// features/user/domain/repositories/i_user_repository.dart
abstract interface class IUserRepository {
  Future<List<User>> fetchUsers();
  Future<User> fetchUserById(String id);
  Future<void> deleteUser(String id);
}
```

---

## Repository Implementation

```dart
// features/user/data/repositories/user_repository.dart
import 'package:dio/dio.dart';
import '../../domain/entities/user.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../datasources/user_remote_data_source.dart';

class UserRepository implements IUserRepository {
  const UserRepository(this._dataSource);

  final UserRemoteDataSource _dataSource;

  @override
  Future<List<User>> fetchUsers() async {
    final dtos = await _dataSource.getUsers();
    return dtos.toEntities();
  }

  @override
  Future<User> fetchUserById(String id) async {
    final dto = await _dataSource.getUserById(id);
    return dto.toEntity();
  }

  @override
  Future<void> deleteUser(String id) async {
    await _dataSource.deleteUser(id);
  }
}
```

---

## Remote Data Source

```dart
// features/user/data/datasources/user_remote_data_source.dart
import 'package:dio/dio.dart';
import '../dtos/user_dto.dart';

class UserRemoteDataSource {
  const UserRemoteDataSource(this._dio);

  final Dio _dio;

  Future<List<UserDto>> getUsers() async {
    final response = await _dio.get<List<dynamic>>('/users');
    return (response.data ?? [])
        .map((json) => UserDto.fromJson(json as Map<String, dynamic>))
        .toList();
  }

  Future<UserDto> getUserById(String id) async {
    final response = await _dio.get<Map<String, dynamic>>('/users/$id');
    return UserDto.fromJson(response.data!);
  }

  Future<void> deleteUser(String id) async {
    await _dio.delete('/users/$id');
  }
}
```

---

## Use Cases

One use case per business operation. Use cases contain no framework code.

```dart
// features/user/domain/use_cases/fetch_users_use_case.dart
import '../entities/user.dart';
import '../repositories/i_user_repository.dart';

class FetchUsersUseCase {
  const FetchUsersUseCase(this._repository);

  final IUserRepository _repository;

  Future<List<User>> execute() => _repository.fetchUsers();
}
```

```dart
// features/user/domain/use_cases/delete_user_use_case.dart
class DeleteUserUseCase {
  const DeleteUserUseCase(this._repository);

  final IUserRepository _repository;

  Future<void> execute(String userId) => _repository.deleteUser(userId);
}
```

---

## Riverpod Wiring for Data Layer

```dart
// features/user/data/providers/user_providers.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/use_cases/fetch_users_use_case.dart';
import '../../domain/use_cases/delete_user_use_case.dart';
import '../datasources/user_remote_data_source.dart';
import '../repositories/user_repository.dart';
import '../../../../core/network/dio_client_provider.dart';

part 'user_providers.g.dart';

@riverpod
UserRemoteDataSource userRemoteDataSource(UserRemoteDataSourceRef ref) {
  return UserRemoteDataSource(ref.watch(dioClientProvider));
}

@riverpod
IUserRepository userRepository(UserRepositoryRef ref) {
  return UserRepository(ref.watch(userRemoteDataSourceProvider));
}

@riverpod
FetchUsersUseCase fetchUsersUseCase(FetchUsersUseCaseRef ref) {
  return FetchUsersUseCase(ref.watch(userRepositoryProvider));
}

@riverpod
DeleteUserUseCase deleteUserUseCase(DeleteUserUseCaseRef ref) {
  return DeleteUserUseCase(ref.watch(userRepositoryProvider));
}
```

---

## Dio Client Setup

```dart
// core/network/dio_client.dart
import 'package:dio/dio.dart';
import '../config/app_config.dart';

Dio createDioClient() {
  final dio = Dio(
    BaseOptions(
      baseUrl: AppConfig.apiBaseUrl,
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 30),
      headers: {'Content-Type': 'application/json'},
    ),
  );

  dio.interceptors.addAll([
    LogInterceptor(
      requestBody: AppConfig.isDev,
      responseBody: AppConfig.isDev,
    ),
    _AuthInterceptor(),
  ]);

  return dio;
}

class _AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // Attach auth token from secure storage if available
    // options.headers['Authorization'] = 'Bearer $token';
    super.onRequest(options, handler);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (err.response?.statusCode == 401) {
      // Handle token refresh or redirect to login
    }
    super.onError(err, handler);
  }
}
```
