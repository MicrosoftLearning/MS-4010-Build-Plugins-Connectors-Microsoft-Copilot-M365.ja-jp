---
lab:
  title: 演習 2 - API プラグイン定義を更新する
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
  description: 次の手順では、API プラグインの定義をアダプティブ カードで更新します。これは、API からのデータをユーザーに表示するために Copilot が使用する必要があります。
  duration: 10 minutes
  level: 200
  islab: true
---

# 演習 2 - API プラグイン定義を更新する

次の手順では、API プラグインの定義をアダプティブ カードで更新します。これは、API からのデータをユーザーに表示するために Copilot が使用する必要があります。

### 演習の期間

- **推定所要時間**: 10 分

## タスク 1 - アダプティブ カードを追加して料理を表示する

Visual Studio Code:

1. **cards/dish.json** ファイルを開き、その内容をコピーします。
1. **appPackage/ai-plugin.json** ファイルを開きます。
1. **functions.getDishes.capabilities.response_semantics** プロパティに、**static_template** という名前の新しいプロパティを追加し、**body** 値を**dish.json** の内容に設定します。
1. 完成したコード スニペットは次のようになります。

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. 変更を保存。

## タスク 2 - アダプティブ カード テンプレートを追加して注文の概要を表示する

Visual Studio Code:

1. **cards/order.json** ファイルを開き、その内容をコピーします。
1. **appPackage/ai-plugin.json** ファイルを開きます。
1. **functions.placeOrder.capabilities.response_semantics** プロパティに、**static_template** という名前の新しいプロパティを追加し、その内容をアダプティブ カードに設定します。
1. 完全なファイルは次のようになります。

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. 変更を保存。
