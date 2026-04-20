---
name: test-gen-enterprise-ja
description: Generates Japanese-style test specifications (テスト仕様書) and test cases from source code or design specs. Use when creating unit test cases (単体テスト仕様書) from Java/TypeScript/C#/VB.NET/COBOL source, generating integration test cases from API specs or DB access code, or producing a テスト仕様書 document required by Japanese SI project standards. Covers equivalence partitioning, boundary value analysis, decision tables, and generates both the test spec document and test code stubs (JUnit/Jest/xUnit/NUnit).
---

# Test Generation — Enterprise (Japanese SI)

> テスト仕様書と単体テストコードを同時に生成する。
> 日本SI標準のドキュメントフォーマットに準拠。

---

## Phase 0: Scope

Ask the user:

> テスト対象を教えてください。
>
> **対象**: クラス名 / メソッド名 / API エンドポイント / バッチ処理名
> **言語**: Java / TypeScript / C# / VB.NET / COBOL
> **テストフレームワーク**: JUnit 5 / Jest / xUnit / NUnit / なし（仕様書のみ）
> **成果物**: テスト仕様書のみ / テストコードのみ / 両方

---

## テスト設計アプローチ

### 同値分割 + 境界値分析

```
【入力: 年齢 (1〜120)】

同値クラス:
  正常: 1〜120
  異常: 0以下 / 121以上 / null / 文字列

境界値:
  下限: 0（異常）/ 1（正常）/ 2（正常）
  上限: 119（正常）/ 120（正常）/ 121（異常）
```

### デシジョンテーブル（複数条件の組み合わせ）

```
条件1: 会員ランク (Gold/Silver/Standard)
条件2: 購入金額 (10,000円以上 / 未満)

| ランク   | 金額    | 割引率 |
|---------|---------|--------|
| Gold    | 以上    | 15%    |
| Gold    | 未満    | 10%    |
| Silver  | 以上    | 8%     |
| Silver  | 未満    | 5%     |
| Standard| 以上    | 3%     |
| Standard| 未満    | 0%     |
```

---

## テスト仕様書テンプレート

```markdown
# 単体テスト仕様書

## テスト対象
| 項目 | 内容 |
|---|---|
| クラス名 | com.example.service.DiscountService |
| メソッド名 | calculateDiscount(rank: String, amount: BigDecimal): BigDecimal |
| 作成日 | YYYY-MM-DD |
| 作成者 | [担当者名] |

## テストケース一覧

| No | テスト区分 | 入力値 | 期待結果 | 観点 |
|---|---|---|---|---|
| T001 | 正常 | rank=Gold, amount=10000 | 1500 | Gold×以上→15% |
| T002 | 正常 | rank=Gold, amount=9999 | 999 | Gold×未満→10% |
| T003 | 正常 | rank=Silver, amount=10000 | 800 | Silver×以上→8% |
| T004 | 境界値 | rank=Gold, amount=10000 | 1500 | 境界下限（以上） |
| T005 | 境界値 | rank=Gold, amount=9999 | 999 | 境界上限（未満） |
| T006 | 異常 | rank=null, amount=10000 | NullPointerException | null入力 |
| T007 | 異常 | rank=Gold, amount=-1 | IllegalArgumentException | 負の金額 |
| T008 | 異常 | rank=UNKNOWN, amount=10000 | IllegalArgumentException | 未定義ランク |
```

---

## テストコード生成（言語別）

### JUnit 5（Java）

```java
@ExtendWith(MockitoExtension.class)
class DiscountServiceTest {

    @InjectMocks
    private DiscountService sut;  // System Under Test

    // T001: Gold × 10,000円以上 → 15%
    @Test
    void calculateDiscount_Gold_AmountAboveThreshold_Returns15Percent() {
        var result = sut.calculateDiscount("Gold", new BigDecimal("10000"));
        assertThat(result).isEqualByComparingTo(new BigDecimal("1500"));
    }

    // T006: null ランク → NullPointerException
    @Test
    void calculateDiscount_NullRank_ThrowsNullPointerException() {
        assertThatThrownBy(() -> sut.calculateDiscount(null, new BigDecimal("10000")))
            .isInstanceOf(NullPointerException.class);
    }

    // T004/T005: 境界値
    @ParameterizedTest
    @CsvSource({
        "9999, 999",   // 境界未満
        "10000, 1500", // 境界以上
    })
    void calculateDiscount_Gold_BoundaryValues(String amount, String expected) {
        var result = sut.calculateDiscount("Gold", new BigDecimal(amount));
        assertThat(result).isEqualByComparingTo(new BigDecimal(expected));
    }
}
```

### Jest（TypeScript）

```typescript
describe("DiscountService.calculateDiscount", () => {
    let sut: DiscountService;
    beforeEach(() => { sut = new DiscountService(); });

    // T001
    it("Gold × 10,000円以上 → 15%割引", () => {
        expect(sut.calculateDiscount("Gold", 10000)).toBe(1500);
    });

    // T006
    it("null ランク → エラー", () => {
        expect(() => sut.calculateDiscount(null, 10000)).toThrow();
    });

    // T004/T005: 境界値
    test.each([
        ["Gold", 9999, 999],
        ["Gold", 10000, 1500],
    ])("境界値: rank=%s, amount=%d → %d", (rank, amount, expected) => {
        expect(sut.calculateDiscount(rank, amount)).toBe(expected);
    });
});
```

### xUnit（C#）

```csharp
public class DiscountServiceTests {
    private readonly DiscountService _sut = new();

    // T001
    [Fact]
    public void CalculateDiscount_GoldRankAboveThreshold_Returns15Percent() {
        var result = _sut.CalculateDiscount("Gold", 10000m);
        Assert.Equal(1500m, result);
    }

    // T004/T005: 境界値
    [Theory]
    [InlineData("Gold", 9999, 999)]
    [InlineData("Gold", 10000, 1500)]
    public void CalculateDiscount_GoldRank_BoundaryValues(
        string rank, decimal amount, decimal expected) {
        Assert.Equal(expected, _sut.CalculateDiscount(rank, amount));
    }
}
```

---

## DB アクセスを含むテスト（リポジトリ層）

```markdown
## リポジトリ単体テスト方針

| アクセス先 | テスト方式 | 理由 |
|---|---|---|
| DB（本番相当） | H2 / Testcontainers | SQL の動作を実際のDBで検証 |
| DB（モック） | Mockito / Jest mock | 速度優先、ロジックのみ検証 |
| 外部API | WireMock / nock | 通信を遮断してレスポンスを制御 |

### 推奨: Testcontainers（Java / TypeScript）
本番と同じDB（PostgreSQL / MySQL / Oracle）をDockerで起動してテスト。
Mockより信頼性が高く、本番障害を防ぐ。
```

---

## Forbidden

- **テストケース番号の欠落禁止** — T001から連番で全件記載
- **境界値の抜け禁止** — 下限-1, 下限, 下限+1 / 上限-1, 上限, 上限+1 を必ず含める
- **ハッピーパスのみ禁止** — 異常系・境界値を必ずセットで生成
- **途中停止禁止** / **AI voice禁止**

---

## Phase E: E2E Test Generation (Playwright)

This phase activates when the user says "Playwright" or "E2E" or "結合テスト自動化".

### Phase E-0: Scope

Ask the user:

> Playwrightテストの対象を教えてください。
>
> **対象URL**: ローカル開発サーバ / ステージング / 本番
> **テスト種別**:
> - `smoke` — 主要ページの表示確認（リンク切れ・404チェック）
> - `form` — フォーム入力・バリデーション・送信フロー
> - `auth` — ログイン・ログアウト・権限チェック
> - `regression` — 既存機能の回帰テスト
> **言語**: TypeScript（推奨）/ JavaScript
> **フレームワーク**: Next.js / React / 静的HTML / その他

### Phase E-1: Test Plan

Generate a test plan table:

```markdown
## E2Eテスト計画

| # | テストID | 対象画面/機能 | テスト種別 | 前提条件 | 期待結果 | 優先度 |
|---|---------|------------|---------|--------|--------|------|
| 1 | E2E-001 | トップページ表示 | smoke | なし | HTTP 200、主要要素表示 | P1 |
| 2 | E2E-002 | ナビゲーションリンク | smoke | なし | 全リンクが正常遷移 | P1 |
```

### Phase E-2: playwright.config.ts Generation

Generate a `playwright.config.ts` tailored to the project:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'mobile', use: { ...devices['iPhone 13'] } },
  ],
});
```

### Phase E-3: Test Code Generation

For each item in the test plan, generate a Playwright test file:

```typescript
// tests/e2e/smoke.spec.ts
import { test, expect } from '@playwright/test';

test.describe('スモークテスト', () => {
  test('トップページが表示される', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveTitle(/AI Agent/);
    await expect(page.locator('nav')).toBeVisible();
  });

  test('ナビゲーションリンクが正常遷移する', async ({ page }) => {
    await page.goto('/');
    const links = page.locator('nav a');
    const count = await links.count();
    for (let i = 0; i < count; i++) {
      const href = await links.nth(i).getAttribute('href');
      if (href && !href.startsWith('http') && !href.startsWith('#')) {
        const response = await page.goto(href);
        expect(response?.status()).toBeLessThan(400);
        await page.goBack();
      }
    }
  });
});
```

### Phase E-4: CI Integration

Generate a GitHub Actions workflow for Playwright:

```yaml
# .github/workflows/playwright.yml
name: Playwright E2E Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Forbidden (E2E固有)

- **実URL直打ち禁止**: `baseURL` を使い、ハードコードしない
- **sleep禁止**: `waitForSelector` / `waitForResponse` を使う
- **XPath禁止**: `getByRole` / `getByText` / `locator('[data-testid]')` を優先
- **テスト間依存禁止**: 各テストは独立して実行できること
