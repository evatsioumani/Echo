# Binary Feature Extraction — Decision Logic

> Αυτό το README εξηγεί αποκλειστικά **πώς εξάγεται κάθε binary feature** από το NLP pipeline. Κάθε διάγραμμα δείχνει τα βήματα απόφασης (negation guards → numeric thresholds → text patterns → default fallback).

**Σύμβολα:**
- 🔵 **NEG** — Negation guard: αν ισχύει, επιστρέφει αμέσως `0`
- 🟢 **NUM** — Numeric threshold: συγκρίνει αριθμητική τιμή με κλινικό threshold
- 🟡 **TXT** — Text pattern: regex αναζήτηση λεκτικών patterns
- 🔴 **DEFAULT** — Fallback αν κανένα προηγούμενο βήμα δεν "έπιασε"

---

## 1. Right Heart

### `pacemaker`
**Section:** RV · FINDINGS · TV

```mermaid
flowchart TD
    A([Normalized text]) --> B{🔵 NEG\n'απινιδωτικό καλώδιο'\nχωρίς 'βηματοδοτ*'?}
    B -->|Ναι| Z0(["`**Label = 0**\n_(ICD ≠ pacemaker)_`"])
    B -->|Όχι| C{🟡 TXT\nMatch 'βηματοδοτ\\w+'\nστο κείμενο;}
    C -->|Κανένα match| Z0b(["`**Label = 0**`"])
    C -->|Match βρέθηκε| D{🔵 Window NEG\n60-char πριν το match:\n'δεν εντοπίζεται / χωρίς';}
    D -->|Negation ανιχνεύθηκε| Z0c(["`**Label = 0**`"])
    D -->|Χωρίς negation| Z1(["`**Label = 1** ✓`"])

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
**Section:** RV | Label=1 μόνο για **μέτρια και άνω** δυσλειτουργία

```mermaid
flowchart TD
    A([Normalized RV text]) --> B{🔵 NEG\n'Φυσιολογική / καλή /\nδιατηρημένη λειτουργία'\nχωρίς αντίθετο qualifier;}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🔵 NEG\n'Ήπια / οριακή'\nεπηρεασμένη\nχωρίς μέτρια/σοβαρή;}
    C -->|Ναι| Z0b(["`**Label = 0**\n_(mild excluded)_`"])
    C -->|Όχι| D{🟡 TXT\n'Μέτρια / σοβαρή\nδυσλειτουργία';}
    D -->|Match| Z1(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟢 NUM\nTAPSE < 15 mm;}
    E -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    E -->|Όχι| F{🟢 NUM\nTV S' < 7.5 cm/s;}
    F -->|Ναι| Z1c(["`**Label = 1** ✓`"])
    F -->|Όχι| G{🟢 NUM\nRV S' < 0.075 m/s;}
    G -->|Ναι| Z1d(["`**Label = 1** ✓`"])
    G -->|Όχι| Z0c(["`**Label = 0**`"])

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
**Section:** RV | Text-only, χωρίς numeric path

```mermaid
flowchart TD
    A([Normalized RV text]) --> B{🔵 NEG\n'Ήπια / μικρού βαθμού\nδιάταση'\nχωρίς 'μετρίου/σοβαρού';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟡 TXT\n'Μετρίου/σοβαρού βαθμού\nδιάταση' ή\n'αυξημένες διαστάσεις/μέγεθος';}
    C -->|Match| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| Z0b(["`**Label = 0**`"])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `right_atrium_dilation`
**Section:** RA | 4 negation guards πριν τα θετικά patterns

```mermaid
flowchart TD
    A([Normalized RA text]) --> B{🔵 NEG\n'Φυσιολογικό μέγεθος /\nφυσιολογικές διαστάσεις /\nεντός ορίων';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🔵 NEG\n'Δεν παρατηρείται /\nχωρίς διάταση';}
    C -->|Ναι| Z0b(["`**Label = 0**`"])
    C -->|Όχι| D{🔵 NEG\n'Μικρού/ήπια/ελαφριά/\nοριακή διάταση\nή αύξηση όγκου';}
    D -->|Ναι| Z0c(["`**Label = 0**`"])
    D -->|Όχι| E{🔵 NEG\n'Ήπια/ελαφρώς/μικρού\nβαθμού αυξημένες\nδιαστάσεις';}
    E -->|Ναι| Z0d(["`**Label = 0**`"])
    E -->|Όχι| F{🟡 TXT\n'Μετρίου/σοβαρού/σημαντικού\nβαθμού διάταση/αύξηση' ή\n'αυξημένο μέγεθος / διατεταμένος';}
    F -->|Match| Z1(["`**Label = 1** ✓`"])
    F -->|Όχι| Z0e(["`**Label = 0**`"])

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
**Section:** LA | Text-only, 3 negation guards

```mermaid
flowchart TD
    A([Normalized LA text]) --> B{🔵 NEG\n'Φυσιολογικό μέγεθος';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🔵 NEG\n'Μικρού βαθμού / ήπια\nαύξηση όγκου ή διάταση';}
    C -->|Ναι| Z0b(["`**Label = 0**`"])
    C -->|Όχι| D{🔵 NEG\n'Ήπια αυξημένες\nδιαστάσεις';}
    D -->|Ναι| Z0c(["`**Label = 0**`"])
    D -->|Όχι| E{🟡 TXT\n'Μέτριου/σοβαρού διάταση',\n'αυξημένο μέγεθος',\n'εμφανίζει διάταση',\n'αύξηση όγκου';}
    E -->|Match| Z1(["`**Label = 1** ✓`"])
    E -->|Όχι| Z0d(["`**Label = 0**`"])

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
    A([Normalized MV text]) --> B{🟡 TXT\nKeyword 'mitra\s*clip';}
    B -->|Match| Z1(["`**Label = 1** ✓`"])
    B -->|Όχι| Z0(["`**Label = 0**`"])

    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `mitral_annular_calcification`
**Section:** MV

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{🔵 NEG\n'Μικρού βαθμού /\nήπια επασβέστωση';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟡 TXT\n'Επασβέστωση μιτροειδικού\nδακτυλίου' / 'ασβεστοποιητική\nμεταβολή' / 'caseous MAC' /\n'επασβεστωμένο μόρφωμα';}
    C -->|Match| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| Z0b(["`**Label = 0**`"])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `mitral_stenosis`
**Section:** MV | Thresholds: ESC 2021

```mermaid
flowchart TD
    A([Normalized MV text]) --> B{🔵 NEG\n'Μικρή/ήπια στένωση'\nχωρίς 'μικρού→μετρίου';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟢 NUM\nMVA ≤ 1.5 cm²;}
    C -->|Ναι| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| D{🟢 NUM\nPHT ≥ 150 ms;}
    D -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟢 NUM\nmeanG ≥ 5 mmHg;}
    E -->|Ναι| Z1c(["`**Label = 1** ✓`"])
    E -->|Όχι| F{🟡 TXT\n'Μέτρια/σοβαρή/\nμικρού→μετρίου\nστένωση';}
    F -->|Match| Z1d(["`**Label = 1** ✓`"])
    F -->|Όχι| Z0b(["`**Label = 0**`"])

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
    A([Normalized MV text]) --> B{🟢 NUM\nERO ≥ 0.20 cm²;}
    B -->|Ναι| Z1(["`**Label = 1** ✓`"])
    B -->|Όχι| C{🟢 NUM\nRVol / MR RV ≥ 30 mL;}
    C -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    C -->|Όχι| D{🟢 NUM\nRF ≥ 30%;}
    D -->|Ναι| Z1c(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟡 TXT\n'Μέτρια/σοβαρή\nανεπάρκεια/διαφυγή';}
    E -->|Match| Z1d(["`**Label = 1** ✓`"])
    E -->|Όχι| Z0(["`**Label = 0**`\n_(ήπια: implicit 0)_"])

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
    A([Normalized AV text]) --> B{🟡 TXT\n'Βιοπροσθετική/μηχανική/\nπροσθετική βαλβίδα'\nή TAVR / TAVI;}
    B -->|Match| Z1(["`**Label = 1** ✓`"])
    B -->|Όχι| Z0(["`**Label = 0**`"])

    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

### `bicuspid_aov`
**Section:** AV | Κρίσιμη διάκριση: φυσική δίπτυχη ≠ μηχανική δίφυλλη

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{🟡 TXT\n'Δίπτυχη/δίφυλλη/δίλοβη/\nbicuspid/δύο πτυχές';}
    B -->|Όχι match| Z0(["`**Label = 0**`"])
    B -->|Match| C{🔵 NEG\nΟπουδήποτε στο κείμενο:\n'μηχανική / προσθετική';}
    C -->|Ναι| Z0b(["`**Label = 0****\n_(δίφυλλη μηχανική ≠ BAV)_`"])
    C -->|Όχι| Z1(["`**Label = 1** ✓\n_(φυσική BAV)_`"])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#FAEEDA,stroke:#854F0B,color:#633806
    style C fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
```

---

### `aortic_stenosis`
**Section:** AV | Thresholds: ESC 2021

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{🔵 NEG\n'Μικρή/ήπια στένωση'\nχωρίς 'μικρού→μετρίου';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟢 NUM\nAVA ≤ 1.5 cm²;}
    C -->|Ναι| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| D{🟢 NUM\nVmax ≥ 3.0 m/s;}
    D -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟢 NUM\nmeanG ≥ 20 mmHg;}
    E -->|Ναι| Z1c(["`**Label = 1** ✓`"])
    E -->|Όχι| F{🟡 TXT\n'Μέτρια/σοβαρή στένωση',\n'Low Flow - Low Gradient';}
    F -->|Match| Z1d(["`**Label = 1** ✓`"])
    F -->|Όχι| Z0b(["`**Label = 0**`"])

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
**Section:** AV | Προσοχή: AR PHT έχει **αντίστροφο** threshold (μικρότερο = χειρότερο)

```mermaid
flowchart TD
    A([Normalized AV text]) --> B{🔵 NEG\n'Μικρή/ήπια ανεπάρκεια'\nοποιαδήποτε σειρά λέξεων\nχωρίς 'μικρού→μετρίου';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟢 NUM\nAR PHT ≤ 500 ms;\n_(αντίστροφο threshold)_}
    C -->|Ναι| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| D{🟢 NUM\nAR ERO ≥ 0.10 cm²;}
    D -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟢 NUM\nAR RVol ≥ 30 mL;}
    E -->|Ναι| Z1c(["`**Label = 1** ✓`"])
    E -->|Όχι| F{🟡 TXT\n'Μέτρια/σοβαρή ανεπάρκεια/\nδιαφυγή' — κανονική +\nαντεστραμμένη σειρά;}
    F -->|Match| Z1d(["`**Label = 1** ✓`"])
    F -->|Όχι| Z0b(["`**Label = 0**`"])

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
**Section:** TV | Text-only, χωρίς numeric path

```mermaid
flowchart TD
    A([Normalized TV text]) --> B{🔵 NEG\n'Μικρή/ήπια ανεπάρκεια'\nοποιαδήποτε σειρά\nχωρίς 'μικρού→μετρίου';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟡 TXT\n'Μέτρια/σοβαρή ανεπάρκεια',\n'μικρού→μετρίου',\n'μετρίου βαθμού τριγλωχινική'\n+ anti-FP guard;}
    C -->|Match| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| Z0b(["`**Label = 0**`"])

    style Z0 fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z0b fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Z1 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style B fill:#E6F1FB,stroke:#3B8BD4,color:#0C447C
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## 5. Αορτή / Περικάρδιο / ΚΚΦ

### `aortic_root_dilation`
**Section:** AORTA | Threshold: ≥ 45 mm

```mermaid
flowchart TD
    A([Normalized AORTA text]) --> B{🟢 NUM Path A\nΤιμές με mm/cm;\nmax value;}
    B -->|Βρέθηκε| C{max ≥ 45 mm;}
    C -->|Ναι| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| Z0(["`**Label = 0**`"])
    B -->|Δεν βρέθηκε| D{🟢 NUM Path B\nStandalone 2-3ψήφιοι\n30–100 αν 'ρίζα/ανιούσα/αορτ';}
    D -->|Βρέθηκε| E{max ≥ 45 mm;}
    E -->|Ναι| Z1b(["`**Label = 1** ✓`"])
    E -->|Όχι| Z0b(["`**Label = 0**`"])
    D -->|Δεν βρέθηκε| F{🟡 TXT\n'Μέτριου/σοβαρού/σημαντικού\nβαθμού διάταση' ή\n'ανεύρυσμα';}
    F -->|Match| Z1c(["`**Label = 1** ✓`"])
    F -->|Όχι| Z0c(["`**Label = 0**`"])

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
**Section:** PERICARDIUM | Threshold: ≥ 20 mm | Masking πλευριτικών/λίπους

```mermaid
flowchart TD
    A([Normalized PERICARDIUM text]) --> M{🔵 MASKING\nΑφαίρεση από κείμενο:\nπλευριτικές συλλογές +\nπερικαρδιακό λίπος\n+ τα mm/cm τους;}
    M --> B{🟢 NUM\nMax mm στο ΚΑΘΑΡΟ κείμενο;}
    B -->|Βρέθηκε| C{max ≥ 20 mm;}
    C -->|Ναι| Z1(["`**Label = 1** ✓`"])
    C -->|Όχι| Z0(["`**Label = 0**`"])
    B -->|Δεν βρέθηκε| D{🔵 NEG\n'Ίχνος / ελεύθερο';}
    D -->|Ναι| Z0b(["`**Label = 0**`"])
    D -->|Όχι| E{🟡 TXT\n'Μεσαίου μεγέθους /\nμεγάλη / μέτριου/σοβαρού\nβαθμού / σημαντική\nπερικαρδιακή';}
    E -->|Match| Z1b(["`**Label = 1** ✓`"])
    E -->|Όχι| Z0c(["`**Label = 0**`"])

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
**Section:** IVC | Threshold: > 21 mm | Αποκλεισμός mmHg

```mermaid
flowchart TD
    A([Normalized IVC text]) --> B{🔵 NEG\n'Φυσιολογικές διαστάσεις'\nχωρίς 'διατεταμένη /\nαυξημένες';}
    B -->|Ναι| Z0(["`**Label = 0**`"])
    B -->|Όχι| C{🟢 NUM\nmm/cm\nαλλά ΟΧΙ mmHg\n_'(?!hg)' lookahead_;}
    C -->|Βρέθηκε > 21 mm| Z1(["`**Label = 1** ✓`"])
    C -->|Βρέθηκε ≤ 21 mm| Z0b(["`**Label = 0**`"])
    C -->|Δεν βρέθηκε| D{🟡 TXT\n'Διατεταμένη'\nοποιαδήποτε σύνταξη;}
    D -->|Match| Z1b(["`**Label = 1** ✓`"])
    D -->|Όχι| E{🟡 TXT\n'Αυξημένες διαστάσεις';}
    E -->|Match| Z1c(["`**Label = 1** ✓`"])
    E -->|Όχι| Z0c(["`**Label = 0**`"])

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

## Κοινά NLP Patterns σε Όλα τα Features

### Accent-Insensitive Regex
Κάθε ελληνικό φωνήεν γράφεται ως character class που καλύπτει τονισμένη **και** άτονη γραφή:

| Class | Αντιστοιχεί σε |
|---|---|
| `[έε]` | `έ` (U+03AD) και `ε` |
| `[ίι]` | `ί` και `ι` |
| `[άα]` | `ά` και `α` |
| `[ήη]` | `ή` και `η` |
| `[όο]` | `ό` και `ο` |
| `[ύυ]` | `ύ` και `υ` |
| `[ώω]` | `ώ` και `ω` |

### Window-based Negation (`_has_negation`)
```
κείμενο: "... δεν εντοπίζεται βηματοδοτ* ..."
                [←  60 chars  →][  match  ]
```
Ελέγχει τα 60 chars **πριν** το match για: `δεν εντοπίζεται`, `δεν παρατηρείται`, `δεν υπάρχει`, `χωρίς`.

### Numeric Unit Handling
```python
val = float(m.group(1).replace(',', '.'))  # κόμμα → τελεία
if unit == 'cm': val *= 10                  # cm → mm
# Αποκλεισμός mmHg: (?!hg) lookahead
```
