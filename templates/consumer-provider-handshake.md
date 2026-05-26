# Consumer-Provider Handshake — Analytics Lab 03

## 1. Thông tin chung

| Mục | Giá trị |
|---|---|
| Consumer | A5 Analytics |
| Provider 1 | A3 Access Gate |
| Provider 2 | A6 Core Business |
| Lab | FIT4110 Lab 03 |
| Mock endpoint | `POST /events` |
| Success response | `202 Accepted` |
| Idempotency | `eventId` |

Analytics là consumer nhận event từ các service khác để tổng hợp metric cho dashboard.

Trong phần Lab 03 này, phạm vi nhóm đang test tập trung vào:

- Access Gate → Analytics
- Core Business → Analytics

IoT và Camera/AI Vision sẽ do thành viên khác trong nhóm bổ sung vào collection tổng nếu cần.

---

## 2. Handshake với Access Gate

### 2.1. Event đã thống nhất

| Mục | Giá trị |
|---|---|
| Pair | Pair 9 |
| Producer | Access Gate |
| Consumer | Analytics |
| Cơ chế thật | Queue async |
| Lab 03 mock | `POST /events` |
| Event type | `access.logs.created` |
| Event version | `1.0.0` |
| Source | `access-gate` |
| Idempotency | `eventId` |

### 2.2. Payload chính thức

```json
{
  "eventId": "evt-20260526-0001",
  "eventType": "access.logs.created",
  "eventVersion": "1.0.0",
  "occurredAt": "2026-05-26T08:00:00Z",
  "source": "access-gate",
  "correlationId": "corr-20260526-0001",
  "data": {
    "logId": "log-7788",
    "cardId": "RFID-2026-001",
    "gateId": "gate-main-01",
    "direction": "IN",
    "status": "ALLOWED"
  }
}

###2.3 Required fields
eventId
eventType
eventVersion
occurredAt
source
correlationId
data.logId
data.cardId
data.gateId
data.direction
data.status

###2.4. Enum
direction = IN, OUT
status = ALLOWED, DENIED, ERROR

### 2.5. Optional fields

Analytics có thể nhận thêm nếu Access Gate có:

data.personId
data.studentId
data.buildingId
data.readerId
data.reasonCode
data.riskLevel

###2.6. Response khi thành công
202 Accepted
{
  "accepted": true,
  "eventId": "evt-20260526-0001",
  "status": "accepted",
  "message": "Event accepted for analytics processing"
}

##2.7. Error response

Nếu thiếu required field:

400 Bad Request
Content-Type: application/problem+json

Nếu sai enum hoặc sai nghiệp vụ:

422 Unprocessable Entity
Content-Type: application/problem+json

###2.8. Metric Analytics tạo từ Access event

Analytics sẽ tổng hợp:

Tổng số lượt access
Số lượt IN
Số lượt OUT
Số lượt ALLOWED
Số lượt DENIED
Số lượt ERROR
Thống kê theo gateId
Thống kê theo khoảng thời gian from/to

3. Handshake với Core Business
3.1. Event đã thống nhất
Mục	Giá trị
Pair	Pair 8
Producer	Core Business
Consumer	Analytics
Cơ chế thật	Queue async
Lab 03 mock	POST /events
Event type	policy.decision.created
Event version	1.0.0
Source	core-business-service
Idempotency	eventId


### 3.2. Payload chính thức
{
  "eventId": "550e8400-e29b-41d4-a716-446655440001",
  "eventType": "policy.decision.created",
  "eventVersion": "1.0.0",
  "occurredAt": "2026-05-21T09:10:00Z",
  "source": "core-business-service",
  "correlationId": "CORR-CORE-POLICY-001",
  "data": {
    "decisionId": "DEC-001",
    "policyId": "POLICY-ACCESS-AFTER-HOURS",
    "subjectId": "SV001",
    "result": "DENIED",
    "reasonCode": "ACCESS_AFTER_HOURS",
    "reasonText": "Access attempt outside allowed time"
  }
}
3.3. Required field
eventId
eventType
eventVersion
occurredAt
source
correlationId
data.decisionId
data.policyId
data.subjectId
data.result
data.reasonCode
data.reasonText

3.4. Enum
result = ALLOWED, DENIED\3.5. Response khi thành công
202 Accepted
{
  "accepted": true,
  "eventId": "550e8400-e29b-41d4-a716-446655440001",
  "status": "accepted",
  "message": "Event accepted for analytics processing"
}

3.6. Metric Analytics tạo từ Core event

Analytics sẽ tổng hợp:

Tổng số policy decision
Số decision ALLOWED
Số decision DENIED
Thống kê theo policyId
Thống kê theo reasonCode
Thống kê theo khoảng thời gian from/to

4. Broker thật

Broker thật chưa chốt trong Lab 03.

Các nội dung sẽ chốt khi tích hợp thật hoặc lab tiếp theo:

RabbitMQ/Kafka/MQTT
host
port
username/password
vhost
routing key
retry policy
dead-letter queue
ack strategy

Nhom dung mock endpoin
POST /events

5. Trạng thái xác nhận
Provider	Event	Trạng thái
A3 Access Gate	access.logs.created	Confirmed
A6 Core Business	policy.decision.created	Confirmed for current mock
A5 Analytics	POST /events, 202 Accepted, Problem error model	Confirmed



