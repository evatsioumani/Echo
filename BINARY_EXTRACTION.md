
# Binary Feature Extraction — Decision Logic


**Step types:**
- NEG — Negation guard: if true, returns `0` immediately
- NUM — Numeric threshold: compares a measured value to a clinical threshold
- TXT — Text pattern: regex search for lexical patterns
- DEFAULT — Fallback if no previous step matched

---

## 1. Right Heart

### `pacemaker`
**Section:** RV - FINDINGS - TV

```mermaid
flowchart TD
    A([Normalized text]) --> B{NEG: ICD lead found\nwithout pacemaker keyword?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{TXT: Match pacemaker keyword}
    C -->|No match| Z0b([Label = 0])
    C -->|Match found| D{Window NEG: 60-char before match\ncontains negation phrase?}
    D -->|Negation found| Z0c([Label = 0])
    D -->|No negation| Z1([Label = 1])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style D fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `rv_systolic_function_depressed`
**Section:** RV — Label=1 only for moderate and above dysfunction

```mermaid
flowchart TD
    A([Normalized RV text]) --> B{NEG: Normal or preserved function\nno opposing qualifier?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NEG: Mild or borderline impaired\nwithout moderate or severe?}
    C -->|Yes| Z0b([Label = 0 mild excluded])
    C -->|No| D{TXT: Moderate or severe dysfunction}
    D -->|Match| Z1([Label = 1])
    D -->|No| E{NUM: TAPSE less than 15 mm}
    E -->|Yes| Z1b([Label = 1])
    E -->|No| F{NUM: TV S-prime less than 7.5 cm per s}
    F -->|Yes| Z1c([Label = 1])
    F -->|No| G{NUM: RV S-prime less than 0.075 m per s}
    G -->|Yes| Z1d([Label = 1])
    G -->|No| Z0c([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1d fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style D fill:#FAEEDA,stroke:#854F0B,color:#633806
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style G fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

---

### `right_ventricle_dilation`
**Section:** RV — Text-only, no numeric path

```mermaid
flowchart TD
    A([Normalized RV text]) --> B{NEG: Mild or minor dilation\nwithout moderate or severe?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{TXT: Moderate or severe dilation\nor increased dimensions or size}
    C -->|Match| Z1([Label = 1])
    C -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `right_atrium_dilation`
**Section:** RA — 4 negation guards before positive patterns

```mermaid
flowchart TD
    A([Normalized RA text]) --> B{NEG: Normal size or\nnormal dimensions or within limits?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NEG: Not observed or no dilation?}
    C -->|Yes| Z0b([Label = 0])
    C -->|No| D{NEG: Minor or mild or\nborderline dilation or volume increase?}
    D -->|Yes| Z0c([Label = 0])
    D -->|No| E{NEG: Mildly or slightly\nincreased dimensions?}
    E -->|Yes| Z0d([Label = 0])
    E -->|No| F{TXT: Moderate or severe dilation\nor enlarged or dilated}
    F -->|Match| Z1([Label = 1])
    F -->|No| Z0e([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0d fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0e fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style D fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style E fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style F fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## 2. Left Atrium

### `left_atrium_dilation`
**Section:** LA — Text-only, 3 negation guards

```mermaid
flowchart TD
    A([Normalized LA text]) --> B{NEG: Normal size?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NEG: Minor or mild\nvolume increase or dilation?}
    C -->|Yes| Z0b([Label = 0])
    C -->|No| D{NEG: Mildly increased dimensions?}
    D -->|Yes| Z0c([Label = 0])
    D -->|No| E{TXT: Moderate or severe dilation\nor increased size or volume increase}
    E -->|Match| Z1([Label = 1])
    E -->|No| Z0d([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0d fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style D fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style E fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## 3. Mitral Valve

### `mitraclip`
**Section:** MV

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{TXT: Keyword mitraclip found?}
    B -->|Match| Z1([Label = 1])
    B -->|No| Z0([Label = 0])

    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `mitral_annular_calcification`
**Section:** MV

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{NEG: Minor or mild calcification?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{TXT: Mitral annular calcification\nor caseous MAC pattern found?}
    C -->|Match| Z1([Label = 1])
    C -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `mitral_stenosis`
**Section:** MV — Thresholds: ESC 2021

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{NEG: Mild stenosis\nwithout mild-to-moderate?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NUM: MVA <= 1.5 cm2}
    C -->|Yes| Z1([Label = 1])
    C -->|No| D{NUM: PHT >= 150 ms}
    D -->|Yes| Z1b([Label = 1])
    D -->|No| E{NUM: meanG >= 5 mmHg}
    E -->|Yes| Z1c([Label = 1])
    E -->|No| F{TXT: Moderate or severe\nor mild-to-moderate stenosis}
    F -->|Match| Z1d([Label = 1])
    F -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1d fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `mitral_regurgitation`
**Section:** MV

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{NUM: ERO >= 0.20 cm2}
    B -->|Yes| Z1([Label = 1])
    B -->|No| C{NUM: RVol or MR RV >= 30 mL}
    C -->|Yes| Z1b([Label = 1])
    C -->|No| D{NUM: RF >= 30 percent}
    D -->|Yes| Z1c([Label = 1])
    D -->|No| E{TXT: Moderate or severe regurgitation}
    E -->|Match| Z1d([Label = 1])
    E -->|No| Z0([Label = 0 mild is implicit 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1d fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## 4. Aortic Valve

### `tavr`
**Section:** AV

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{TXT: Bioprosthetic or mechanical\nor prosthetic valve or TAVR or TAVI?}
    B -->|Match| Z1([Label = 1])
    B -->|No| Z0([Label = 0])

    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `bicuspid_aov`
**Section:** AV — Natural bicuspid vs mechanical bileaflet

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{TXT: Bicuspid or bileaflet\nor bilobed or two cusps?}
    B -->|No match| Z0([Label = 0])
    B -->|Match| C{NEG: Anywhere in text\nmechanical or prosthetic?}
    C -->|Yes| Z0b([Label = 0 mechanical bileaflet not BAV])
    C -->|No| Z1([Label = 1 native BAV])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
    style C fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
```

---

### `aortic_stenosis`
**Section:** AV — Thresholds: ESC 2021

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{NEG: Mild stenosis\nwithout mild-to-moderate?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NUM: AVA <= 1.5 cm2}
    C -->|Yes| Z1([Label = 1])
    C -->|No| D{NUM: Vmax >= 3.0 m per s}
    D -->|Yes| Z1b([Label = 1])
    D -->|No| E{NUM: meanG >= 20 mmHg}
    E -->|Yes| Z1c([Label = 1])
    E -->|No| F{TXT: Moderate or severe stenosis\nor Low Flow Low Gradient}
    F -->|Match| Z1d([Label = 1])
    F -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1d fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `aortic_regurgitation`
**Section:** AV — AR PHT has an inverse threshold: lower value = worse regurgitation

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{NEG: Mild regurgitation any word order\nwithout mild-to-moderate?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NUM: AR PHT <= 500 ms inverse threshold}
    C -->|Yes| Z1([Label = 1])
    C -->|No| D{NUM: AR ERO >= 0.10 cm2}
    D -->|Yes| Z1b([Label = 1])
    D -->|No| E{NUM: AR RVol >= 30 mL}
    E -->|Yes| Z1c([Label = 1])
    E -->|No| F{TXT: Moderate or severe regurgitation\nforward and reversed word order}
    F -->|Match| Z1d([Label = 1])
    F -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1d fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `tricuspid_valve_regurgitation`
**Section:** TV — Text-only, no numeric path

```mermaid
flowchart TD
    A([Normalized TV text]) --> B{NEG: Mild regurgitation any word order\nwithout mild-to-moderate?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{TXT: Moderate or severe regurgitation\nor mild-to-moderate plus anti-FP guard}
    C -->|Match| Z1([Label = 1])
    C -->|No| Z0b([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## 5. Aorta / Pericardium / IVC

### `aortic_root_dilation`
**Section:** AORTA — Threshold: >= 45 mm

```mermaid
flowchart TD
    A([Normalized AORTA text]) --> B{NUM Path A: Values with mm or cm - track max}
    B -->|Found| C{max >= 45 mm?}
    C -->|Yes| Z1([Label = 1])
    C -->|No| Z0([Label = 0])
    B -->|Not found| D{NUM Path B: Standalone 2-3 digit numbers\n30 to 100 if aorta context present}
    D -->|Found| E{max >= 45 mm?}
    E -->|Yes| Z1b([Label = 1])
    E -->|No| Z0b([Label = 0])
    D -->|Not found| F{TXT: Moderate or severe or significant\ndilation or aneurysm}
    F -->|Match| Z1c([Label = 1])
    F -->|No| Z0c([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `pericardial_effusion`
**Section:** PERICARDIUM — Threshold: >= 20 mm — Masks pleural effusions and fat

```mermaid
flowchart TD
    A([Normalized PERICARDIUM text]) --> M{MASKING: Remove pleural effusions\nand pericardial fat with their mm/cm values}
    M --> B{NUM: Find max mm value in clean text}
    B -->|Found| C{max >= 20 mm?}
    C -->|Yes| Z1([Label = 1])
    C -->|No| Z0([Label = 0])
    B -->|Not found| D{NEG: Trace or free?}
    D -->|Yes| Z0b([Label = 0])
    D -->|No| E{TXT: Medium or large or moderate\nor severe pericardial effusion}
    E -->|Match| Z1b([Label = 1])
    E -->|No| Z0c([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style M fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style B fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style E fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `dilated_ivc`
**Section:** IVC — Threshold: > 21 mm — Excludes mmHg values

```mermaid
flowchart TD
    A([Normalized IVC text]) --> B{NEG: Normal dimensions\nwithout dilated or increased?}
    B -->|Yes| Z0([Label = 0])
    B -->|No| C{NUM: mm or cm but NOT mmHg\nusing lookahead to exclude hg}
    C -->|Found > 21 mm| Z1([Label = 1])
    C -->|Found <= 21 mm| Z0b([Label = 0])
    C -->|Not found| D{TXT: Dilated - any syntax}
    D -->|Match| Z1b([Label = 1])
    D -->|No| E{TXT: Increased dimensions}
    E -->|Match| Z1c([Label = 1])
    E -->|No| Z0c([Label = 0])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0c fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1b fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z1c fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#FAEEDA,stroke:#854F0B,color:#633806
    style E fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## Common NLP Patterns

### Accent-Insensitive Regex
Every Greek vowel is written as a character class matching both accented and unaccented forms:

| Class | Matches |
|---|---|
| `[έε]` | accented and unaccented e |
| `[ίι]` | accented and unaccented i |
| `[άα]` | accented and unaccented a |
| `[ήη]` | accented and unaccented i (eta) |
| `[όο]` | accented and unaccented o |
| `[ύυ]` | accented and unaccented u |
| `[ώω]` | accented and unaccented o (omega) |

### Window-based Negation
Checks the 60 characters **before** a keyword match for negation phrases like "not found" or "without".

### Numeric Unit Handling
```python
val = float(m.group(1).replace(',', '.'))  # comma to dot decimal
if unit == 'cm': val *= 10                  # convert cm to mm
# Exclude mmHg using (?!hg) lookahead
```
