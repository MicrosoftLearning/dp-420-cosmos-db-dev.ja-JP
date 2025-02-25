---
title: オンラインでホストされる手順
permalink: index.html
layout: home
---

このリポジトリには、Microsoft のコース「[DP-420 Microsoft Azure Cosmos DB を使用するクラウドネイティブ アプリケーションの設計と実装][course-description]」のハンズオン ラボ演習と、同等の [Microsoft Learn のマイペースで進められるモジュール][learn-collection]が含まれています。 演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

> &#128221; この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 [https://azure.microsoft.com][azure] で無料試用版にサインアップできます。

# ラボ

## C# ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign csharp_setup_labs = "" | split: "," %} {% assign csharp_regular_labs = "" | split: "," %} {% assign genai_setup_labs = "" | split: "," %} {% assign genai_python_labs = "" | split: "," %} {% assign genai_javascript_labs = "" | split: "," %}

{% for activity in labs %} {% assign segments = activity.url | split: "/" %}

  {% if segments[1] == "instructions" and segments.size == 3 %} {% if activity.lab.module contains "Setup" %} {% assign csharp_setup_labs = csharp_setup_labs | push: activity %} {% else %} {% assign csharp_regular_labs = csharp_regular_labs | push: activity %} {% endif %}
  
  {% elsif activity.url contains '/gen-ai/python/instructions' %} {% assign genai_python_labs = genai_python_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/javascript/instructions' %} {% assign genai_javascript_labs = genai_javascript_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/common/instructions' and activity.lab.module contains "Setup" %} {% assign genai_setup_labs = genai_setup_labs | push: activity %} {% endif %} {% endfor %}

---

### **セットアップ ラボ**

| モジュール | ラボ |
| --- | --- |
{% for activity in csharp_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **ラボ **

| モジュール | ラボ |
| --- | --- |
{% for activity in csharp_regular_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

## **生成 AI ラボ**

### **一般的なセットアップ ラボ**

| モジュール | ラボ |
| --- | --- |
{% for activity in genai_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **JavaScript ラボ**

| モジュール | ラボ |
| --- | --- |
{% for activity in genai_javascript_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Python ラボ**

| モジュール | ラボ |
| --- | --- |
{% for activity in genai_python_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
