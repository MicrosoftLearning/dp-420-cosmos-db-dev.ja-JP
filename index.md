---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
ms.openlocfilehash: 13dd011c620f0d260b29a807eb919d7922e3e8df
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024974"
---
このリポジトリには、Microsoft のコース「[DP-420 Microsoft Azure AI ソリューションの設計と実装][course-description]」の実践的な演習と、同等の [Microsoft Learn のマイペースで進められるモジュール][learn-collection]が含まれています。 演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

> &#128221; この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 講師からサブスクリプションが提供されていない場合は、[https://azure.microsoft.com][azure] で無料の試用版にサインアップできます。

## <a name="labs"></a>ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| モジュール | ラボ |
| --- | --- |
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
