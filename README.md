# Cerberus Tokenizer

Ein BPE-Tokenizer, den ich als privates Lernprojekt entwickelt habe. Cerberus ist auf hohe Kompression für Deutsch, Englisch, Code und JSON ausgelegt und schlägt dabei mehrere bekannte Open-Source-Tokenizer.

> Kein professionelles Produkt, kein Ersatz für GPT-4o oder Llama. Ich bin 16 und das hier ist ein Hobbyprojekt.

---

## Was ist das?

Ich wollte verstehen, wie Tokenizer eigentlich funktionieren, und hab angefangen, einen von Grund auf selbst zu bauen. Herausgekommen ist Cerberus: ein 65k-Vocab BPE-Tokenizer mit Byte-Fallback, der vor allem für deutsche Texte besser komprimiert als die meisten frei verfügbaren Alternativen.

Ob das praktisch nützlich ist, weiß ich nicht. Für mich war der Lerneffekt der Punkt.

---

## Benchmark-Ergebnisse

### Kompression, Bytes per Token

| Tokenizer | Vocab | B/T DE | B/T EN | B/T Code | B/T JSON |
|---|---|---|---|---|---|
| **Cerberus** | 65,024 | **4.486** | **4.495** | **3.513** | **2.973** |
| GPT-2 | 50,257 | 2.714 | 4.086 | 2.455 | 2.307 |
| Falcon-7B | 65,024 | 3.557 | 4.068 | 3.223 | 2.636 |
| CodeGen | 50,295 | 2.714 | 4.124 | 3.127 | 2.602 |
| GPT-NeoX | 50,277 | 3.255 | 4.162 | 3.303 | 2.745 |
| OPT | 50,265 | 2.714 | 4.086 | 2.455 | 2.307 |
| Mistral | 32,000 | 3.298 | 4.068 | 3.096 | 2.529 |

Cerberus liegt in allen vier Kategorien vorne. Das war nicht von Anfang an so, hat einige Iterationen gebraucht.

### OOD-German (Out-of-Domain)

Getestet auf Domänen, die nicht im Trainings-Corpus waren:

| Domäne | B/T |
|---|---|
| Tech/German | 3.949 |
| Juristisch | 5.378 |
| Medizinisch | 4.260 |

---

## Konfiguration

| Parameter | Wert |
|---|---|
| Algorithmus | BPE mit Byte-Fallback |
| Vocab-Größe | 65,024 |
| BPE-Merges | 64,960 |
| Special/Reserved Tokens | 64 |
| Pre-Tokenizer | GPT-4o-Stil |
| MAX_TOKEN_LENGTH | 10 |
| MIN_FREQUENCY | 5 |
| BPE-Dropout (Training) | 0.10 |
| Targeted Augmentation | True (aug_rate 0.005) |
| Trainings-Corpus | Raw, kein Preprocessing |

### Trainings-Corpus-Mix

| Sprache/Domäne | Zeilen |
|---|---|
| Deutsch | 10,000,000 |
| Englisch | 8,000,000 |
| Code | 15,000,000 |
| Tech | 3,000,000 |

---

## Was ich dabei gelernt habe

**Corpus nicht bereinigen.** Ich hab viel Zeit damit verschwendet, den Trainings-Corpus zu deduplizieren und zu filtern, in der Hoffnung, dass saubere Daten bessere Tokens ergeben. Das Gegenteil war der Fall: BPE braucht Wiederholungen, um starke Merges zu lernen. Bereinigter Corpus hat die Kompression konsistent verschlechtert.

**Pre-Tokenizer macht einen Unterschied.** Ein GPT-4o-ähnlicher Pre-Tokenizer hat vor allem die JSON-Kompression leicht verbessert, durch präzisere Tokenisierung von Zahlensequenzen.

---

## Bekannte Probleme

- **9 bad_fragments** (d, hab, werd, verarbei, arbie, klien, tokk), die der Tokenizer strukturell schlecht verarbeitet. Nur via Vocab-Pruning lösbar, hab ich noch nicht angegangen.
- **Whitespace-Inkonsistenz bei deutschen Texten** durch den Raw-Corpus, in der Praxis kaum spürbar.
- Weitere Kompressionsverbesserungen stoßen mit Standard-BPE an eine Grenze.

---

## Ausgabedateien

| Datei | Beschreibung |
|---|---|
| `output/cerberus_tokenizer.json` | Inferenz-Tokenizer |
| `output/cerberus_tokenizer_train.json` | Training-Tokenizer mit BPE-Dropout |
