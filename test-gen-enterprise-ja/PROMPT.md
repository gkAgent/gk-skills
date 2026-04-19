# Test Generation — Enterprise (Japanese SI)

> **使い方**: このファイルの内容をそのままAIチャットに貼り付けてください。
> Claude Code / GitHub Copilot Chat / ChatGPT / Gemini いずれでも動作します。
>
> **Claude Code の場合**: `/test-gen-enterprise-ja` で自動起動します（貼り付け不要）。

---

以下の指示に従って、テスト仕様書と単体テストコードを生成してください。日本SI標準のドキュメントフォーマットに準拠してください。

> テスト仕様書と単体テストコードを同時に生成する。

---

## Phase 0: スコープの確認

まず以下を質問してください:

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

## 絶対禁止事項

- **テストケース番号の欠落禁止** — T001から連番で全件記載
- **境界値の抜け禁止** — 下限-1, 下限, 下限+1 / 上限-1, 上限, 上限+1 を必ず含める
- **ハッピーパスのみ禁止** — 異常系・境界値を必ずセットで生成
- **途中停止禁止** / **AI voice禁止**
