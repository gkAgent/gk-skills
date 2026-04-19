# API 仕様書生成 — OpenAPI 3.0

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/api-spec-openapi` で自動起動します（貼り付け不要）。

---

以下の指示に従って、既存コードから OpenAPI 3.0 仕様書を生成してください。

---

## Phase 0: スコープの確認

まず以下を質問してください:

> 対象コードを教えてください。
>
> **言語/FW**: Spring Boot / ASP.NET Core / NestJS / Express / Next.js App Router
> **認証方式**: Bearer JWT / API Key / OAuth2 / セッション / なし
> **出力形式**: YAML（デフォルト）/ JSON
> **既存仕様書**: なし（新規生成）/ あり（差分更新）

---

## OpenAPI 3.0 基本構造

```yaml
openapi: "3.0.3"
info:
  title: "社員管理API"
  version: "1.0.0"
  description: "社員・部署情報の CRUD API"

servers:
  - url: "https://api.example.com/v1"
    description: "本番環境"
  - url: "http://localhost:8080/v1"
    description: "ローカル開発"

security:
  - bearerAuth: []

paths:
  /employees:
    get:
      summary: "社員一覧取得"
      operationId: "listEmployees"
      tags: ["employees"]
      parameters:
        - name: deptId
          in: query
          required: false
          schema: { type: integer }
          description: "部署IDで絞り込み"
        - name: page
          in: query
          schema: { type: integer, default: 1 }
        - name: size
          in: query
          schema: { type: integer, default: 20, maximum: 100 }
      responses:
        "200":
          description: "成功"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/EmployeeListResponse"
        "401":
          $ref: "#/components/responses/Unauthorized"

    post:
      summary: "社員新規登録"
      operationId: "createEmployee"
      tags: ["employees"]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateEmployeeRequest"
            example:
              empName: "田中 太郎"
              deptId: 10
              salary: 450000
      responses:
        "201":
          description: "作成成功"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Employee"
        "400":
          $ref: "#/components/responses/BadRequest"
        "422":
          $ref: "#/components/responses/UnprocessableEntity"

  /employees/{empId}:
    get:
      summary: "社員単件取得"
      operationId: "getEmployee"
      tags: ["employees"]
      parameters:
        - name: empId
          in: path
          required: true
          schema: { type: integer, format: int64 }
      responses:
        "200":
          description: "成功"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Employee"
        "404":
          $ref: "#/components/responses/NotFound"

components:
  schemas:
    Employee:
      type: object
      required: [empId, empName, deptId, salary, hireDate, isActive]
      properties:
        empId:
          type: integer
          format: int64
          example: 100001
        empName:
          type: string
          maxLength: 40
          example: "田中 太郎"
        deptId:
          type: integer
          example: 10
        salary:
          type: number
          format: double
          description: "月給（円）"
          example: 450000
        hireDate:
          type: string
          format: date
          example: "2020-04-01"
        isActive:
          type: boolean
          example: true

    CreateEmployeeRequest:
      type: object
      required: [empName, deptId, salary]
      properties:
        empName:
          type: string
          maxLength: 40
        deptId:
          type: integer
        salary:
          type: number
          minimum: 0

    EmployeeListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: "#/components/schemas/Employee"
        total:
          type: integer
          example: 150
        page:
          type: integer
          example: 1
        size:
          type: integer
          example: 20

    ErrorResponse:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          example: "VALIDATION_ERROR"
        message:
          type: string
          example: "入力値が不正です"
        details:
          type: array
          items:
            type: object
            properties:
              field: { type: string }
              message: { type: string }

  responses:
    Unauthorized:
      description: "認証エラー"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            code: "UNAUTHORIZED"
            message: "認証が必要です"
    BadRequest:
      description: "リクエスト不正"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
    NotFound:
      description: "リソース未検出"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            code: "NOT_FOUND"
            message: "指定された社員が存在しません"
    UnprocessableEntity:
      description: "バリデーションエラー"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

---

## 言語別コードからの変換パターン

### Spring Boot @RestController → OpenAPI

```java
@RestController
@RequestMapping("/api/employees")
@RequiredArgsConstructor
public class EmployeeController {

    @GetMapping("/{empId}")
    public ResponseEntity<EmployeeDto> get(@PathVariable Long empId) { ... }

    @PostMapping
    public ResponseEntity<EmployeeDto> create(@RequestBody @Valid CreateEmployeeRequest req) { ... }
}
```

変換規則:
- `@GetMapping("/{id}")` → `GET /path/{id}` + path parameter
- `@RequestBody @Valid` → `requestBody: required: true` + スキーマ
- `ResponseEntity<T>` → `responses: "200": schema: $ref`
- `@Valid` + Bean Validation → `required` / `minimum` / `maxLength` 等

### Next.js App Router → OpenAPI

```typescript
// app/api/employees/[empId]/route.ts
export async function GET(req: Request, { params }: { params: { empId: string } }) {
    const employee = await getEmployee(Number(params.empId));
    if (!employee) return Response.json({ error: "Not found" }, { status: 404 });
    return Response.json(employee);
}
```

---

## OpenAPI 生成の鉄則

```
全エンドポイントで必ず記載する項目:
□ operationId — SDK 生成時のメソッド名になる。必ず一意で意味のある名前
□ 全レスポンスコード — 200 だけでなく 400/401/403/404/422/500 を網羅
□ example — スキーマの example は必ず実データに近い値
□ description — 日本語で業務的な説明（技術説明ではなく）
□ required — 必須フィールドの明示（省略禁止）
```

---

## 絶対禁止事項

- **operationId の省略禁止** — SDK 生成・テスト自動化で必須
- **エラーレスポンスの 200 のみ記載禁止** — 401/404/422 を最低限記載
- **型の `any` / `object` のみ記載禁止** — プロパティを必ず展開する
- **途中停止禁止** / **AI voice禁止**
