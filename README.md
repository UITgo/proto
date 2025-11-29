# UITGo Proto Definitions

Thư mục chứa các gRPC/Protobuf definitions cho inter-service communication trong UITGo.

## Tổng quan

UITGo sử dụng **gRPC** cho internal service communication. Các proto files định nghĩa contracts giữa các services, đảm bảo type safety và versioning.

## Proto Files

### `user.proto`
Định nghĩa User Service gRPC interface:

```protobuf
service UserService {
  rpc GetProfile(GetProfileRequest) returns (GetProfileResponse);
}

message GetProfileRequest {
  string user_id = 1;
}

message GetProfileResponse {
  bool exists = 1;
  string user_id = 2;
  string name = 3;
  string avatar_url = 4;
  string role = 5;
}
```

**Sử dụng bởi:**
- `trip-command-service`: Gọi `GetProfile` để verify user role

### `trip.proto`
Định nghĩa Trip Service gRPC interface (nếu có):

```protobuf
// Trip proto definitions (nếu có)
```

**Sử dụng bởi:**
- Các service cần gọi trip operations qua gRPC (nếu có)

### `driverstream.proto`
Định nghĩa Driver Stream Service gRPC interface (legacy, có thể deprecated):

```protobuf
// Driver stream proto definitions (nếu có)
```

**Lưu ý**: Driver Stream hiện tại chủ yếu sử dụng HTTP REST API, gRPC có thể được dùng cho internal calls.

## Cách generate code từ proto

### Node.js (NestJS services)

```bash
# Cài đặt dependencies
npm install @grpc/grpc-js @grpc/proto-loader

# Generate TypeScript types (nếu có tool)
# Hoặc sử dụng proto-loader để load dynamic
```

### Go (driver-stream)

```bash
# Generate Go code từ proto
protoc --go_out=. --go_opt=paths=source_relative \
  --go-grpc_out=. --go-grpc_opt=paths=source_relative \
  proto/user.proto
```

## Sử dụng trong services

### NestJS Service (trip-command-service)

```typescript
// Load proto file
import * as protoLoader from '@grpc/proto-loader';
import * as grpc from '@grpc/grpc-js';

const packageDefinition = protoLoader.loadSync(
  path.join(__dirname, '../../proto/user.proto'),
  { keepCase: true, longs: String, enums: String, defaults: true }
);

const userProto = grpc.loadPackageDefinition(packageDefinition).user as any;
const client = new userProto.UserService(
  'user-service:50051',
  grpc.credentials.createInsecure()
);
```

### Go Service (driver-stream)

```go
// Import generated code
import pb "github.com/UITGo/proto/driverstream"

// Use in service
```

## Versioning

Proto files nên được versioned để đảm bảo backward compatibility:

- **v1**: Current version
- **v2**: Future breaking changes (nếu có)

## Best Practices

1. **Backward Compatibility**: Không xóa fields, chỉ thêm optional fields mới
2. **Naming**: Sử dụng snake_case cho message fields
3. **Documentation**: Thêm comments cho mỗi service và message
4. **Versioning**: Tag proto files với version numbers

## Xem thêm

- **gRPC Documentation**: https://grpc.io/docs/
- **Protobuf Guide**: https://developers.google.com/protocol-buffers
- **Service READMEs**: Xem README của từng service để biết cách sử dụng proto
