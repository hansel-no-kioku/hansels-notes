# MkDocs+

[MkDocs+](http://bwmarrin.github.io/MkDocsPlus/)

## js-sequence-diagrams

[js-sequence-diagrams](https://bramp.github.io/js-sequence-diagrams/)

```markdown
勇者さん->武器屋さん: どうのつるぎ下さい
Note right of 武器屋さん: 考え中
武器屋さん-->勇者さん: 1000Gになります
勇者さん->>武器屋さん: いりません
{: .diagram }
```

勇者さん->武器屋さん: どうのつるぎ下さい
Note right of 武器屋さん: 考え中
武器屋さん-->勇者さん: 1000Gになります
勇者さん->>武器屋さん: いりません
{: .diagram }

## mermaid

[mermaid](https://mermaidjs.github.io/)

## フローチャート

[Flowchart](https://mermaidjs.github.io/flowchart.html) みた方が早い。

```markdown
graph LR
  I(わたし) -- 愛 --> You[あなた]
{: .mermaid }
```

graph LR
  I(わたし) -- 愛 --> You[あなた]
{: .mermaid }

## シーケンス図

[Sequence diagram](https://mermaidjs.github.io/sequenceDiagram.html) みた方が早い。

```markdown
sequenceDiagram
  クライアント->>サーバ: やって～
  activate サーバ
  サーバ-->>クライアント: やったよ～
{: .mermaid }
```

sequenceDiagram
  クライアント->>サーバ: やって～
  activate サーバ
  サーバ-->>クライアント: やったよ～
{: .mermaid }

## ガントチャート

[Gant diagrams](https://mermaidjs.github.io/gantt.html) みた方が早い。

あまり使わなそうなので省略。
