---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
ms.openlocfilehash: 4a255765ba2dc3584dcbe24b5a7bf2f5e5ef9f85
ms.sourcegitcommit: b59f546be1101bc48880d7b368897090bdb8f9ec
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/08/2022
ms.locfileid: "147866803"
---
このリポジトリには、Microsoft のコース「[DP-420 Microsoft Azure AI ソリューションの設計と実装][course-description]」の実践的な演習と、同等の [Microsoft Learn のマイペースで進められるモジュール][learn-collection]が含まれています。 演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

> &#128221; この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 [https://azure.microsoft.com][azure] で無料試用版にサインアップできます。

## <a name="labs"></a>ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| モジュール | ラボ |
| --- | --- |
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
