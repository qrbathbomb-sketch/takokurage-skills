---
name: "event-time-schedule"
description: "イベントの施工・本番・撤去スケジュールを、Excelの時間軸の表に移動できる図形を乗せる形で作る・更新する。図形と時間の目盛りのズレを自動チェックする。"
---

# イベント施工・本番スケジュール（Excel・図形タイプ）

縦が時間軸、横が担当（全体・クライアント・代理店・自社メンバー・運営・警備・リース・美術・テクニカル・デコなど）の表。各作業は**セルではなく図形**にして表の上に乗せる。現場の人がドラッグですぐ動かせるようにするためだ。

## いちばん大事なこと：図形と時間の目盛りをズラさない

ユーザーが一番気にしている点。過去に、手で置いた図形がまとめて30分下にズレていたことがある（例：「23:00-3:30」の図形が0:00-4:00の位置にあった）。次のルールを必ず守る。

1. **図形は手で置かない。** 位置は必ず `build_schedule.py` が開始・終了時刻から計算する。
2. **時間の行はすべて同じ高さにする。** 1行＝30分、28.5pt。行の高さが1行でも違うと、そこから下の図形が全部ズレる。
3. **時刻のラベルは、その時刻の線の上に置く。** 「9:00」は9:00の行の上端の線に揃える（上揃え）。
4. **翌日の時刻は24を足して書く。** 設定ファイルでは、10/29から見た翌3:30は `"27:30"`、翌9:00は `"33:00"`。シートの範囲外の時刻はエラーで止まるので、止まったら時刻の書き方を直す。
5. **1つの図形には作業を1つだけ入れる。** 「6:30-7:00 着替え / 7:00-8:00 オリエン」のように作業を並べると、図形の端と書いてある時刻がずれる。作業ごとに図形を分ける。ただし「本番中（午前）」のように、ざっくりした枠に補足を書くのはよい。
6. **図形の1行目には、その図形の範囲どおりの時刻を書く**（例：`9:00-12:00`）。チェックはこの1行目を見る。ほかの日から続く分は `（前日19:00〜）〜7:30` と書き、カッコの中はチェックの対象外になる。
7. **作ったら必ず `verify_schedule.py` にかける。** 「ズレなし」になるまで直す。最後にLibreOfficeで画像にして、目でも確かめる。
8. **ユーザーからもらったExcelを更新するときは、先に `verify_schedule.py` にかける。** 元の図形がズレていないか確かめ、ズレていれば「書いてある時刻が正しい」として図形を置き直す。どれを直したかはユーザーに伝える。

## 書き方のルール

- **シートの分け方**：日ごとにシートを分ける（例：`10.29-30 施工` / `10.30 施工` / `10.31 本番` / `11.1 本番・撤去`）。夜通しの日は、翌朝までを1枚に入れる。
- **24時を過ぎる日**：最後の日（撤去日）は `over24: true` にして、時刻を「26:00」のように書く。タイトルにも「※26:00＝11/2 2:00」と書いておく。
- **入り・出の時刻**：入り時刻は `▼15:00IN`（type `in`、ラベルの下端が入り時刻）、出の時刻は `▲23:30OUT`（type `out`）で表す。
- **自社の立ち合い**：全員で立ち会う必要はない。1人ずつ「午前／午後」「夜」のように、ざっくり分けてよい。誰がやるか決まっていなければ、「※担当・時間は仮」と書いておく。
- **本番中**：細かく割らず、「10:00-18:00 本番時間」のようにざっくりでよい。
- **受け取り**：弁当・リース・備品チャーターなどの受け取りは、**ユーザー本人の列**に図形で入れる（他の人の担当と言われたら、そちらの列に入れる）。
- **決まっていないこと**：推測で埋めずに「確認中」と書く。「相談中」「調整中」「仮」も含め、これらの言葉が入った行は自動で赤字になる。
- **手配・相談事項**（シーバー・道路使用許可・リース期間・警備の相談など）は、TIMEの列より右に置くメモ（type `memo`）にまとめる。メモは印刷範囲の外になる。
- **パワポなど別の資料から写すとき**：図形の位置からどの列・何時かを読み取る。読み取れないもの、資料どうしで食い違うものは、勝手に決めずにユーザーに聞く。

## 手順

1. もらった資料を読む（Excel・パワポ・テキストのメモ）。Excelに図形があれば、`verify_schedule.py` で元のズレを確認する。
2. 分からないことは、作り始める前にまとめて質問する（Claudeなら AskUserQuestion）。たとえば次のようなこと。
   - どの資料を元にするか
   - シートの分け方
   - 作業がいつ終わるか
   - 最終日を何時までにするか
3. 下の2つのスクリプトを作業フォルダに書き出す。
4. 設定ファイル `spec.json` を作る（形式は下にある）。
5. `python3 build_schedule.py spec.json out.xlsx` で作る。
6. `python3 verify_schedule.py out.xlsx` でズレを確かめる。「ズレなし」になるまで設定ファイルを直して、作り直す。
7. `soffice --headless --convert-to pdf out.xlsx` と `pdftoppm -r 70 -png out.pdf p` で画像にして、各シートを目で確かめる。文字があふれていないか、図形が重なっていないかを見る。
8. ファイル名を `MMDD_案件名_施工スケジュール.xlsx` にして、ユーザーに渡す（Claudeなら SendUserFile）。返信では次のことを短く伝える。
   - 何を反映したか
   - 「確認中」として赤字にした項目
   - 自分で解釈したところ
   - 確認はLibreOfficeでしたので、Excelで開くと見え方が少し違う可能性があること

## spec.json の形

```json
{
  "slot_minutes": 30,
  "row_height_pt": 28.5,
  "column_width": 20,
  "columns": [
    {"group": "全体"},
    {"group": "クライアント様", "name": "ご担当者", "key": "client"},
    {"group": "自社", "name": "メンバーB", "key": "B"},
    {"group": "自社", "name": "メンバーA", "key": "A"},
    {"group": "警備"}
  ],
  "sheets": [
    {
      "name": "10.29-30 施工",
      "title": "施工スケジュール／10月29日(木)〜10月30日(金)",
      "start": "14:00", "end": "37:00",
      "items": [
        {"col": "A", "type": "in", "time": "15:00", "text": "▼15:00IN"},
        {"col": "A", "start": "15:00", "end": "17:00", "text": "15:00-17:00\n自社倉庫で積み込み"},
        {"col": "全体", "start": "21:00", "end": "33:00", "text": "21:00-翌9:00\n搬入・設営"},
        {"col": "client", "start": "33:00", "end": "36:00", "part": "left", "text": "9:00-12:00\n展示ショーケース"},
        {"col": "client", "start": "33:00", "end": "36:00", "part": "right", "text": "9:00-12:00\n検収"},
        {"col": "B", "to_col": "A", "start": "33:00", "end": "34:00", "color": "yellow", "text": "9:00-10:00 クライアントチェック"},
        {"col": "A", "type": "out", "time": "23:30", "text": "▲23:30OUT"},
        {"type": "memo", "row": 0, "lines": ["【手配・相談事項】", "■シーバー", "・10/29夜から必要か 確認中"]}
      ]
    },
    {"name": "11.1 本番・撤去", "title": "…※26:00＝11/2 2:00", "start": "6:00", "end": "29:00", "over24": true, "items": []}
  ]
}
```

- `columns` の `key` は、`items` の `col` で使う名前。同じ `group` が続く列は、見出しの段が結合される。
- `start` は「ちょうど何時」にする。`end` は30分単位。
- `part` は `full`（初期値）、`left`、`right` のどれか。同じ列に同じ時間の作業が2つあるときは、左右に分けて並べる。
- `color` は `gray`（初期値）、`blue`、`yellow`、`green`、`pink`、`white` のどれか、または6桁の色コード。
- 15分しかない作業は図形が細くなるので、文字を短くする（例：`20:45 警備オリエン`）。
- 雛形のExcelをもらっても、そこへ図形を書き足すのではなく、列の構成（担当と名前）を写して、このスクリプトで作り直す。

## build_schedule.py

```python
"""Build a time-grid production schedule (.xlsx) whose tasks are movable shapes.

usage: python3 build_schedule.py spec.json out.xlsx
Every shape position is computed from its start/end time with the same row height
that the grid uses, so shapes always sit on the time ruler.
"""
import json, sys, zipfile, datetime
from xml.sax.saxutils import escape
import openpyxl
from openpyxl.styles import Font, Alignment, Border, Side, PatternFill
from openpyxl.utils import get_column_letter as CL

spec = json.load(open(sys.argv[1], encoding='utf-8'))
OUT = sys.argv[2]

FONT = spec.get('font', 'Meiryo UI')
SLOT = spec.get('slot_minutes', 30)          # minutes per grid row
ROW_PT = spec.get('row_height_pt', 28.5)     # every time row has exactly this height
ROW_EMU = round(ROW_PT * 12700)
COL_W = spec.get('column_width', 20)         # Excel width units for task columns
FIRST = 4                                    # first time row (rows 1-3: title + 2 header rows)
TIME_L, FIRST_COL = 2, 3                     # B = TIME, task columns start at C
PAD = 40000                                  # EMU gap left/right inside a column
RED_WORDS = spec.get('red_words', ['確認中', '相談中', '調整中', '仮'])
FILL = {'gray': 'F2F2F2', 'blue': 'DDEBF7', 'yellow': 'FFF2CC', 'green': 'E2EFDA', 'pink': 'FCE4D6', 'white': 'FFFFFF'}

# columns: [{"group": "自社", "name": "メンバーB", "key": "B"}, {"group": "全体", "key": "全体"}, ...]
cols = spec['columns']
KEY = {c.get('key', c.get('name') or c['group']): FIRST_COL + i for i, c in enumerate(cols)}
LAST_COL = FIRST_COL + len(cols) - 1
TIME_R = LAST_COL + 1


def mins(s):
    h, m = str(s).replace('：', ':').split(':'); return int(h) * 60 + int(m)


def col_emu(width):  # Excel char width -> EMU (Excel: px = trunc(w*7+5), 1px = 9525 EMU)
    return int(width * 7 + 5) * 9525

thin = Side(style='thin'); med = Side(style='medium'); hair = Side(style='hair', color='A6A6A6')
wb = openpyxl.Workbook(); wb.remove(wb.active)
drawings = []

for sh in spec['sheets']:
    ws = wb.create_sheet(sh['name'][:31])
    t0, t1 = mins(sh['start']), mins(sh['end'])        # may exceed 24:00 (e.g. 29:00)
    assert (t1 - t0) % SLOT == 0 and t0 % 60 == 0, 'start must be on the hour, end on a slot'
    over24 = sh.get('over24', False)
    nrows = (t1 - t0) // SLOT
    per_hour = 60 // SLOT

    widths = {TIME_L: 9, TIME_R: 9}
    for i, c in enumerate(cols):
        widths[FIRST_COL + i] = c.get('width', COL_W)
    ws.column_dimensions['A'].width = 2
    for ci, w in widths.items():
        ws.column_dimensions[CL(ci)].width = w

    # title
    ws.cell(1, TIME_L, sh['title']).font = Font(name=FONT, size=14, bold=True)
    ws.row_dimensions[1].height = 24
    # header (row 2 = group, row 3 = person)
    hdr = PatternFill('solid', fgColor='F2F2F2')
    for r in (2, 3):
        ws.row_dimensions[r].height = 18
        for ci in range(TIME_L, TIME_R + 1):
            c = ws.cell(r, ci); c.fill = hdr
            c.font = Font(name=FONT, size=10, bold=True)
            c.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
            c.border = Border(left=thin, right=thin, top=med if r == 2 else thin, bottom=thin if r == 2 else med)
    for ci in (TIME_L, TIME_R):
        ws.cell(2, ci, 'TIME'); ws.merge_cells(start_row=2, start_column=ci, end_row=3, end_column=ci)
    i = 0
    while i < len(cols):
        g = cols[i]['group']; j = i
        while j + 1 < len(cols) and cols[j + 1]['group'] == g: j += 1
        ws.cell(2, FIRST_COL + i, g)
        for k in range(i, j + 1):
            ws.cell(3, FIRST_COL + k, cols[k].get('name', ''))
        if j > i:
            ws.merge_cells(start_row=2, start_column=FIRST_COL + i, end_row=2, end_column=FIRST_COL + j)
        if j == i and not cols[i].get('name'):
            ws.merge_cells(start_row=2, start_column=FIRST_COL + i, end_row=3, end_column=FIRST_COL + i)
        i = j + 1

    # time rows: identical heights; hour line = thin, other slots = hair
    for k in range(nrows):
        r = FIRST + k
        ws.row_dimensions[r].height = ROW_PT
        on_hour = (k % per_hour == 0)
        for ci in range(TIME_L, TIME_R + 1):
            ws.cell(r, ci).border = Border(
                left=med if ci == TIME_L else thin, right=med if ci == TIME_R else thin,
                top=(thin if on_hour else hair) if ci not in (TIME_L, TIME_R) or on_hour else None,
                bottom=med if k == nrows - 1 else None)
        if on_hour:
            m = t0 + k * SLOT
            h = m // 60
            for ci in (TIME_L, TIME_R):
                c = ws.cell(r, ci)
                if over24:
                    c.value = f'{h}:00'
                else:
                    c.value = datetime.time(h % 24, 0); c.number_format = 'h:mm'
                # label hugs the top line of its hour = the ruler mark
                c.font = Font(name=FONT, size=11, bold=True)
                c.alignment = Alignment(horizontal='center', vertical='top')
    # closing label under the last row
    endc = ws.cell(FIRST + nrows, TIME_L, f'{t1 // 60}:00' if over24 else f'{(t1 // 60) % 24}:00')
    endc.font = Font(name=FONT, size=11, bold=True); endc.alignment = Alignment(horizontal='center', vertical='top')

    ws.freeze_panes = ws.cell(FIRST, FIRST_COL)
    ws.page_setup.orientation = 'landscape'; ws.page_setup.paperSize = ws.PAPERSIZE_A3
    ws.sheet_properties.pageSetUpPr.fitToPage = True
    ws.page_setup.fitToWidth = 1; ws.page_setup.fitToHeight = 1
    ws.print_area = f'{CL(TIME_L)}1:{CL(TIME_R)}{FIRST + nrows}'

    # ---------- shapes ----------
    col_x = {}; x = 0
    for ci in range(1, TIME_R + 12):
        col_x[ci] = x; x += col_emu(ws.column_dimensions[CL(ci)].width or 8.43)
    row_y0 = sum(round(ws.row_dimensions[r].height * 12700) for r in (1, 2, 3))

    def anchor_y(m):
        assert t0 <= m <= t1, f'{sh["name"]}: {m // 60}:{m % 60:02d} is outside {sh["start"]}-{sh["end"]}'
        d = m - t0
        k, rest = divmod(d, SLOT)
        return FIRST - 1 + k, round(rest / SLOT * ROW_EMU), row_y0 + round(d / SLOT * ROW_EMU)

    def anchor_x(key, part):
        ci = KEY[key]; w = col_emu(widths[ci])
        a, b = {'full': (0, 1), 'left': (0, .5), 'right': (.5, 1)}[part]
        x1 = round(w * a) + (PAD if a == 0 else PAD // 2)
        x2 = round(w * b) - (PAD if b == 1 else PAD // 2)
        return ci - 1, x1, x2, col_x[ci]

    def runs(text, sz, algn, memo=False):
        out = []
        for line in text.split('\n'):
            color = 'C00000' if any(w in line for w in RED_WORDS) else '000000'
            b = ' b="1"' if (memo and line[:1] in '【■') or (not memo and line[:1] in '▼▲') else ''
            r = (f'<a:r><a:rPr lang="ja-JP" altLang="en-US" sz="{sz}"{b}><a:solidFill><a:srgbClr val="{color}"/></a:solidFill>'
                 f'<a:latin typeface="{FONT}"/><a:ea typeface="{FONT}"/></a:rPr><a:t>{escape(line)}</a:t></a:r>') if line else ''
            out.append(f'<a:p><a:pPr algn="{algn}"/>{r}<a:endParaRPr lang="ja-JP" sz="{sz}"/></a:p>')
        return ''.join(out)

    parts = []
    sid = 2
    for it in sh.get('items', []):
        typ = it.get('type', 'box')
        if typ == 'memo':
            c1 = TIME_R + 1; c2 = c1 + it.get('span_cols', 6)
            r1 = FIRST - 1 + it.get('row', 0); r2 = r1 + max(6, len(it['lines']) + 2)
            xa, ya = col_x[c1 + 1] + 100000, row_y0 + it.get('row', 0) * ROW_EMU
            cx = col_x[c2 + 1] - col_x[c1 + 1]; cy = (r2 - r1) * ROW_EMU
            frm = (c1, 100000, r1, 0); to = (c2, 0, r2, 0)
            sp = (f'<xdr:sp macro="" textlink=""><xdr:nvSpPr><xdr:cNvPr id="{sid}" name="メモ {sid}"/><xdr:cNvSpPr/></xdr:nvSpPr>'
                  f'<xdr:spPr><a:xfrm><a:off x="{xa}" y="{ya}"/><a:ext cx="{cx}" cy="{cy}"/></a:xfrm><a:prstGeom prst="rect"><a:avLst/></a:prstGeom>'
                  f'<a:solidFill><a:srgbClr val="FFF2CC"/></a:solidFill><a:ln><a:noFill/></a:ln></xdr:spPr>'
                  f'<xdr:txBody><a:bodyPr vertOverflow="clip" lIns="144000" tIns="108000" rIns="144000" bIns="108000" anchor="t"/><a:lstStyle/>'
                  f'{runs(chr(10).join(it["lines"]), 1600, "l", memo=True)}</xdr:txBody></xdr:sp>')
        else:
            part = it.get('part', 'full')
            ci0, x1, x2, cx0 = anchor_x(it['col'], part)
            ci1 = KEY[it.get('to_col', it['col'])] - 1
            if typ == 'box':
                m1, m2 = mins(it['start']), mins(it['end'])
                assert m2 > m1, f'{it}'
            elif typ == 'in':          # ▼ label: bottom edge = arrival time
                m2 = mins(it['time']); m1 = max(t0, m2 - SLOT)
            elif typ == 'out':         # ▲ label: top edge = leave time
                m1 = mins(it['time']); m2 = min(t1, m1 + SLOT)
            r1, y1, ay1 = anchor_y(m1); r2, y2, ay2 = anchor_y(m2)
            if ci1 != ci0:
                _, _, x2, _ = anchor_x(it['to_col'], 'full')
            frm = (ci0, x1, r1, y1); to = (ci1, x2, r2, y2)
            xa = cx0 + x1; cxx = col_x[ci1 + 1] + x2 - xa
            if typ == 'box':
                color = FILL.get(it.get('color', 'gray'), it.get('color', 'F2F2F2'))
                sp = (f'<xdr:sp macro="" textlink=""><xdr:nvSpPr><xdr:cNvPr id="{sid}" name="作業 {sid}"/><xdr:cNvSpPr/></xdr:nvSpPr>'
                      f'<xdr:spPr><a:xfrm><a:off x="{xa}" y="{ay1}"/><a:ext cx="{cxx}" cy="{ay2 - ay1}"/></a:xfrm><a:prstGeom prst="rect"><a:avLst/></a:prstGeom>'
                      f'<a:solidFill><a:srgbClr val="{color}"/></a:solidFill><a:ln w="6350"><a:solidFill><a:srgbClr val="000000"/></a:solidFill></a:ln></xdr:spPr>'
                      f'<xdr:txBody><a:bodyPr wrap="square" lIns="36000" tIns="0" rIns="36000" bIns="0" anchor="ctr"/><a:lstStyle/>'
                      f'{runs(it["text"], it.get("size", 1000), "ctr")}</xdr:txBody></xdr:sp>')
            else:
                sp = (f'<xdr:sp macro="" textlink=""><xdr:nvSpPr><xdr:cNvPr id="{sid}" name="IN/OUT {sid}"/><xdr:cNvSpPr txBox="1"/></xdr:nvSpPr>'
                      f'<xdr:spPr><a:xfrm><a:off x="{xa}" y="{ay1}"/><a:ext cx="{cxx}" cy="{ay2 - ay1}"/></a:xfrm><a:prstGeom prst="rect"><a:avLst/></a:prstGeom><a:noFill/><a:ln><a:noFill/></a:ln></xdr:spPr>'
                      f'<xdr:txBody><a:bodyPr wrap="square" lIns="36000" tIns="0" rIns="36000" bIns="0" anchor="{"b" if typ == "in" else "t"}"/><a:lstStyle/>'
                      f'{runs(it["text"], 1100, "l")}</xdr:txBody></xdr:sp>')
        f = (f'<xdr:twoCellAnchor><xdr:from><xdr:col>{frm[0]}</xdr:col><xdr:colOff>{frm[1]}</xdr:colOff><xdr:row>{frm[2]}</xdr:row><xdr:rowOff>{frm[3]}</xdr:rowOff></xdr:from>'
             f'<xdr:to><xdr:col>{to[0]}</xdr:col><xdr:colOff>{to[1]}</xdr:colOff><xdr:row>{to[2]}</xdr:row><xdr:rowOff>{to[3]}</xdr:rowOff></xdr:to>'
             f'{sp}<xdr:clientData/></xdr:twoCellAnchor>')
        parts.append(f); sid += 1
    drawings.append('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n<xdr:wsDr xmlns:xdr="http://schemas.openxmlformats.org/drawingml/2006/spreadsheetDrawing" '
                    'xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">' + ''.join(parts) + '</xdr:wsDr>')

tmp = OUT + '.grid.xlsx'
wb.save(tmp)

# inject drawings
zin = zipfile.ZipFile(tmp)
files = {n: zin.read(n) for n in zin.namelist()}
ct = files['[Content_Types].xml'].decode()
for i, d in enumerate(drawings, 1):
    files[f'xl/drawings/drawing{i}.xml'] = d.encode()
    ct = ct.replace('</Types>', f'<Override PartName="/xl/drawings/drawing{i}.xml" ContentType="application/vnd.openxmlformats-officedocument.drawing+xml"/></Types>')
    files[f'xl/worksheets/_rels/sheet{i}.xml.rels'] = (
        '<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">'
        f'<Relationship Id="rIdDr1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/drawing" Target="../drawings/drawing{i}.xml"/></Relationships>').encode()
    x = files[f'xl/worksheets/sheet{i}.xml'].decode()
    if 'xmlns:r=' not in x.split('>', 2)[1]:
        x = x.replace('<worksheet ', '<worksheet xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" ', 1)
    x = x.replace('</worksheet>', '<drawing r:id="rIdDr1"/></worksheet>', 1)
    files[f'xl/worksheets/sheet{i}.xml'] = x.encode()
files['[Content_Types].xml'] = ct.encode()
with zipfile.ZipFile(OUT, 'w', zipfile.ZIP_DEFLATED) as z:
    for n, b in files.items():
        z.writestr(n, b)
import os; os.remove(tmp)
print('saved', OUT)
```

## verify_schedule.py

```python
"""Check that every shape on a time-grid schedule sits where its own text says.

usage: python3 verify_schedule.py schedule.xlsx [--time-col B] [--first-row 4]
Reads the time labels in the TIME column to know which minute each row starts at,
converts each shape's anchor (row + rowOff) into minutes, and compares it with the
first "H:MM" (and the end time of "H:MM-H:MM" / "H:MM〜H:MM") written in the shape.
"""
import sys, re, zipfile, datetime, argparse, posixpath
import openpyxl
from openpyxl.utils import column_index_from_string

ap = argparse.ArgumentParser()
ap.add_argument('xlsx'); ap.add_argument('--time-col', default='B'); ap.add_argument('--first-row', type=int, default=4)
ap.add_argument('--tol', type=int, default=2, help='allowed difference in minutes')
a = ap.parse_args()

wb = openpyxl.load_workbook(a.xlsx)
z = zipfile.ZipFile(a.xlsx)
names = z.namelist()
wbx = z.read('xl/workbook.xml').decode()
wbrels = z.read('xl/_rels/workbook.xml.rels').decode()
rid_target = dict(re.findall(r'Id="([^"]+)"[^>]*Target="([^"]+)"', wbrels)) | {k: v for v, k in re.findall(r'Target="([^"]+)"[^>]*Id="([^"]+)"', wbrels)}
sheets = re.findall(r'<sheet [^>]*name="([^"]+)"[^>]*r:id="([^"]+)"', wbx)

TIME = re.compile(r'(\d{1,2})\s*[:：]\s*(\d{2})')


def label_minutes(v):
    if v is None: return None
    if isinstance(v, datetime.time): return v.hour * 60 + v.minute
    if isinstance(v, datetime.datetime): return v.hour * 60 + v.minute
    if isinstance(v, datetime.timedelta): return int(v.total_seconds() // 60)
    if isinstance(v, (int, float)): return round(v * 1440)
    m = TIME.search(str(v));
    return int(m.group(1)) * 60 + int(m.group(2)) if m else None


def fmt(m): return f'{(m // 60) % 24}:{m % 60:02d}'

problems = 0
for sname, rid in sheets:
    ws = wb[sname]
    target = rid_target[rid].lstrip('/')
    spath = target if target.startswith('xl/') else 'xl/' + target
    relp = posixpath.join(posixpath.dirname(spath), '_rels', posixpath.basename(spath) + '.rels')
    if relp not in names: continue
    rels = z.read(relp).decode()
    dr = re.search(r'Target="([^"]*drawings/[^"]+)"', rels)
    if not dr: continue
    dpath = posixpath.normpath(posixpath.join(posixpath.dirname(spath), dr.group(1)))
    dxml = z.read(dpath).decode()

    # --- row -> minute map from the TIME column (labels may skip rows, day may wrap) ---
    tc = column_index_from_string(a.time_col)
    labels = []
    prev = None; day = 0
    for r in range(a.first_row, ws.max_row + 1):
        m = label_minutes(ws.cell(r, tc).value)
        if m is None: continue
        if prev is not None and m + day < prev: day += 1440
        labels.append((r, m + day)); prev = m + day
    if len(labels) < 2:
        print(f'[{sname}] TIME列にラベルが見つからない'); continue
    slot = (labels[1][1] - labels[0][1]) / (labels[1][0] - labels[0][0])  # minutes per row
    r0, m0 = labels[0]
    for (ra, ma), (rb, mb) in zip(labels, labels[1:]):
        if abs((mb - ma) / (rb - ra) - slot) > 0.01:
            print(f'[{sname}] TIME列の刻みが不揃い: 行{ra}={fmt(ma)} 行{rb}={fmt(mb)}'); problems += 1

    def row_h(r):
        h = ws.row_dimensions[r].height
        return (h if h else 15) * 12700  # EMU

    def to_min(row0, off):  # row0 is 0-based
        r = row0 + 1
        return m0 + (r - r0) * slot + off / row_h(r) * slot

    grid_rows = 0  # time rows = consecutive rows with the same height as the first one
    while ws.row_dimensions[r0 + grid_rows].height == ws.row_dimensions[r0].height and r0 + grid_rows <= ws.max_row:
        grid_rows += 1
    heights = {ws.row_dimensions[r].height for r in range(r0, labels[-1][0])}
    if len(heights) > 1:
        print(f'[{sname}] 時間行の高さが不揃い {sorted(h or 15 for h in heights)} → 図形の位置がズレる原因'); problems += 1

    n = 0
    for anc in re.findall(r'<xdr:twoCellAnchor.*?</xdr:twoCellAnchor>', dxml, re.S):
        c1, o1, rr1, ro1, c2, o2, rr2, ro2 = map(int, re.findall(r'<xdr:(?:col|colOff|row|rowOff)>(-?\d+)', anc)[:8])
        text = '\n'.join(''.join(re.findall(r'<a:t>(.*?)</a:t>', p)) for p in re.findall(r'<a:p>.*?</a:p>', anc, re.S))
        if rr1 + 1 < r0: continue
        top, bot = to_min(rr1, ro1), to_min(rr2, ro2)
        if ro1 > row_h(rr1 + 1) + 1 or ro2 > row_h(rr2 + 1) + 1:
            print(f'[{sname}] rowOffが行の高さを超えている: {text[:20]!r}'); problems += 1
        first_line = text.strip().split('\n')[0]
        if first_line.startswith('【'): continue  # メモ枠は対象外
        is_in = first_line.startswith('▼'); is_out = first_line.startswith('▲')
        # 時刻が書かれた最初の行だけを見る（1図形=1作業がルール）
        line = next((re.sub(r'[（(][^）)]*[）)]', '', l) for l in text.split('\n')
                     if TIME.search(re.sub(r'[（(][^）)]*[）)]', '', l))), None)
        if line is None: continue
        n += 1
        ts = [int(h) * 60 + int(m) for h, m in TIME.findall(line)]
        ends_only = re.match(r'\s*[〜～~ー－-]\s*\d', line) is not None
        rng = re.search(r'(\d{1,2})\s*[:：]\s*(\d{2})\s*[-~〜～ー－]\s*(?:翌)?(\d{1,2})\s*[:：]\s*(\d{2})', line)
        sheet_top, sheet_end = m0, m0 + grid_rows * slot

        def off(x, y):
            d = (x - y) % 1440; return min(d, 1440 - d)
        bad = []
        if ends_only:
            e = ts[0]
            if abs(bot - sheet_end) > a.tol and off(bot, e) > a.tol:
                bad.append(f'終了 {fmt(e)} のはずが {fmt(round(bot))} の位置')
        else:
            got = bot if is_in else top
            if off(got, ts[0]) > a.tol and not (abs(top - sheet_top) <= a.tol and not is_in):
                bad.append(f'開始 {fmt(ts[0])} のはずが {fmt(round(got))} の位置')
            if rng and not (is_in or is_out):
                e = int(rng.group(3)) * 60 + int(rng.group(4))
                if off(bot, e) > a.tol and abs(bot - sheet_end) > a.tol:
                    bad.append(f'終了 {fmt(e)} のはずが {fmt(round(bot))} の位置')
        if bad:
            problems += 1
            col = openpyxl.utils.get_column_letter(c1 + 1)
            print(f'[{sname}] {col}列 {text.replace(chr(10), " / ")[:50]!r}: ' + '、'.join(bad))
    print(f'[{sname}] 時刻入り図形 {n}件をチェック（1行={slot:g}分）')
print('ズレなし' if problems == 0 else f'要確認 {problems}件')
sys.exit(1 if problems else 0)
```