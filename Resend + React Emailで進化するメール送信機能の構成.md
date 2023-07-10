---
marp: true
theme: default
class: invert
---

# 💌 Resend + React Emailで進化するメール送信機能の構成

<style scoped>
section { 
    font-size: 28px; 
}
</style>

2023.07.12
tatti ([@tachibanayu24](https://twitter.com/tachibanayu24))

---

## 👋 はじめに

- 自前でSMTPサーバーを用意せずにemail送信の機能を実装したい場合は、Amazon SESやSendGridなどのSaaSを利用するのが一般的
- しかしSendGridなどファーストチョイスになるようなSaaSには課題も
- Resend + React Emailでモダンなメール送信機能を構築できるのでは

---

## 💭 SendGridについて

- SendGridはクラウドベースのSMTPプロバイダで、メールサーバの維持管理の手間を省ける
- しかし、無料プランでは1ユーザーしか利用できず、2FAが必須、メール送信ログの閲覧が限定的、UIがもっさりしていて遅いなどの問題が
- 料金も高い気がする（専用IPは1万円以上のプランから）

---

## 🎈 Resendの紹介

- Resend(会社)は2023年に創業したサンフランシスコのベンチャー企業
  - Y Combinatorからシード調達
  - 創業メンバーはWorkOSのコア開発メンバー
- Resend(サービス)は、スパムリストに入れられた場合はすぐに通知、容易に構築可能、大規模トランザクションに対応するスケーラビリティなどの特徴がある
- シンプルなダッシュボード([demo](https://resend.com/overview))

---

## 💸 Resendのプラン

![h:600](https://firebasestorage.googleapis.com/v0/b/orenotion.appspot.com/o/pages%2FScreenshot%202023-07-09%20at%2019.05.48.png?alt=media&token=1e4474d4-6224-4c4e-9038-cfcf6f1c5cca)

---

## 💌 React Emailの紹介

- React EmailはReactベースでメール本文などを構成できるOSSで、Resendのメンバーが開発(まだベータ版)
- ButtonやLinkなどのコンポーネントが用意されており、これにstyleを当てて構築
- render APIでReactをHTML文字列にパースできるので、サービス画面で「こういうメールが送信されますよ」とプレビューするのが用意
- Reactなのでcssやレイアウトの共通化も当然できる
- どのようなメールが送信されるかを一覧できるStorybookのようなカタログがビルトインされていてテスト送信も可能！
  - [sample](https://demo.react.email/preview/vercel-invite-user)

---

## ✨ サンプル

React Emailのリポジトリにあるサンプルコードを見てみよう

---

### メール本文

<style scoped>
section { 
    font-size: 28px; 
}

pre {
  font-size: 20px;
}
</style>

 - Reactなのでメール本文の変数は普通にpropsで定義
 - styleも自由に当てられる(tailwind, emotion, ...)
 - Preview(Gmailなどでメール一覧で表示される部分)など、メールの較正に最適化されたコンポーネントが提供されている

```tsx
export const WaitlistEmail: React.FC<Readonly<WaitlistEmailProps>> = ({
  name,
}) => (
  <Html>
    <Head />
    <Preview>Thank you for joining our waitlist and for your patience</Preview>
    <Body style={main}>
      <Container style={container}>
        <Heading style={h1}>Coming Soon.</Heading>
        <Text style={text}>
          Thank you {name} for joining our waitlist and for your patience. We
          will send you a note when we have something new to share.
        </Text>
      </Container>
    </Body>
  </Html>
);
```

---

### メール送信部分

<style scoped>
section { 
    font-size: 28px; 
}
</style>

- SDKを利用してペイロードを詰めるだけ
- `react`というプロパティにコンポーネントをそのまま突っ込める
- サーバがReactと別のところにある場合は、 `render`APIでパースしたHTMLを`html`というプロパティに詰めて送信すればOK

```ts
const send = async (req: NextApiRequest, res: NextApiResponse) => {
  const data = await resend.sendEmail({
    from: "bu@resend.dev",
    to: "delivered@resend.dev",
    subject: "Waitlist",
    react: WaitlistEmail({ name: "Bu" }),
  });
  return res.status(200).send(data);
};
```

---

## 👋 おわり

ResendとReact Emailを利用してメール送信機能を構成する場合、以下のようなものが得られます。

- 優れたUIによる送受信の分析
- フルマネージドIPアドレス、最適化されたネットワークアーキテクチャ、スケーラビリティ、スパム対策
- 完全にコードベースで制御されたメールテンプレート
- StorybookのようなUIによるメールのカタログ化と送信テスト

ただしとても若いサービスなのでそこは留意して選定する必要があります。
