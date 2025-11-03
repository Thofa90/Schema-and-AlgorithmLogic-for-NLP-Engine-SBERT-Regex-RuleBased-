# Task: Schema and Algorithm Logic for NLP Engine

## 🧠 SBERT-Based NLP Query Parser for Housing Search

This project implements a natural-language query understanding engine to extract structured housing search parameters from user input. It uses SBERT-based semantic similarity, fallback logic, and canonical mapping to output a structured parameter list in JSON format.

##  Key Techniques Used

| Step | Technique |
|------|-----------|
| Synonym Expansion | Manual synonym map and canonical normalization |
| Intent Detection | SBERT-based cosine similarity |
| Value Extraction | Regex for numbers, keywords (price, room, floor) |
| Multi-parameter Handling | Controlled via MULTI_PARAMS list |
| Fallback Logic | Word-level fuzzy match with canonical normalization |
| Post-Processing | Deduplication, list formatting, conflict resolution |

---

## 🔍 Objective

Transform raw user queries (e.g., "Looking for a 2-room apartment under 1000€ with balcony in neukölln") into structured JSON parameters like:

```json
[
  {"priority":1, "parameter": "room_requirement", "value": 2, "operator": ">="},
  {"priority":2,"parameter": "price_constraint", "value": 1000, "operator": "<="},
  {"priority":3,"parameter": "feature_request", "value": ["balcony"], "operator": "=="},
  {"priority":4,"parameter": "housing_type", "value": ["apartment"], "operator": "=="},
  {"priority":5, "parameter": "district", "value": "Neukölln", "operator": "==" },
]
```

---

## 📦 Core Components (Processing Pipeline)

### 1. **User Query Input**
- **Input:** Raw natural-language query (string)
- **Example:** `"Need a 3-room flat with garden in Kreuzberg under 1200 euros"`

---

### 2. **Text Preprocessing**
- Normalize to lowercase
- Append synonyms using `synonym_map`
- Resulting query is expanded for matching

---

### 3. **SBERT Encoding**
- SBERT (`all-MiniLM-L6-v2`) encodes the expanded query.
- Embeddings are compared with template embeddings via cosine similarity.

---

### 4. **Semantic Phrase Matching**
- **Similarity ≥ threshold (0.15)**: Match is accepted.
- If match found:
  - Parameter is identified.
  - Regex is used to extract numerical values (e.g., rent, rooms, floor).
  - Canonical value normalization is applied.The extracted value from a user query is converted into a standard, consistent format, also known as its canonical form.This ensures that different ways of expressing the same idea are treated identically in the structured output.

---

### 5. **Fallback Phrase Matching**
- If SBERT fails:
  - Query is matched word-by-word against example phrases in `parameter_templates`.
  - Matches are added to grouped parameters using canonical mapping.

---

### 6. **Parameter Grouping**
- Multi-value parameters (`MULTI_PARAMS`) are collected as lists.
- Others are stored as single values.

---

### 7. **Post-Processing**
- Duplicate values removed.
- Conflict resolution (e.g., no "garden" if "green area" is present).
- Canonical forms enforced for multi-values.

---

### 8. **Structured JSON Output**
- Parameters are sorted by priority and output in structured format.
- Operator logic: `<=`, `>=`, `==` based on parameter.

---

## 📋 Parameter Templates

These define semantic patterns used for similarity matching. Each parameter has multiple example phrases.The `parameter_templates` dictionary defines the possible intent categories and their representative example phrases. These serve as the *backbone* for semantic matching using SBERT.

Each parameter (e.g., `price_constraint`, `feature_request`, `environment_preference`) contains a list of example phrases that represent how users might express their preferences in natural language. These templates are embedded using SBERT and compared to the input query to find the most semantically relevant matches.

This makes the system **flexible** to user phrasing and **extendable** by simply adding more examples to any parameter category.

**Example (partial):**
```python
parameter_templates = {
    "price_constraint": ["rent under 1000", "maximum 1200 euros"..],
    "room_requirement": ["3-room apartment", "2 rooms apartment", "2 rooms flat"..],
    "furnishing": ["furnished apartment", "includes furniture"..],
    "feature_request": ["has balcony", "balcony", "garden", "lift", "garage", "parking space", "storage room", "backyard"..],
    "Education":["school","university","childcare","daycare","near kindergartens","near colleges","near schools"..],
    "environment_preference": ["quiet", "peaceful area","regional statistics","safe district","green", "quiet neighborhood", "near forest", "near park"..],
    "health_safety": ["near hospitals","near clinics","vet clinics","clinics","dental officies","near pharmacies","crime statistics","crime free","safe area"..],
    "convenience_access": ["supermarkets", "grocery stores", "banks", "ATMs", "post office", "laundromats", "dry cleaners","shopping centers", "convenience stores", "bakeries", 
    "cafes", "pools","swimming pools","restaurants", "libraries", "gyms", "fitness centers","recreational zones","playground","pet stores"..],
    "location_proximity": ["close to metro", "near subway", "train station", "by bus stop", "near bus station", "public transportation", "bikelanes","near transit", "close to transportation", "walkable to station"..],
    "city_filter": ["in Berlin"],
    "entertainment_option": ["theaters","social clubs","venues","convention hall","night life","night clubs","cultural density","near bar","activities"..],
    "housing_type": ["house", "flat", "flat share","apartment", "shared apartment"..],
    "contract_type": ["overnight stay", "temporary contract","limited contract", "unlimited contract", "long-term rent"..],
    "living_area_m2": ["at least 50 square meters", "minimum 40 m2", "over 60m2", "100 square meters", "50m2", "60 m2"..],
    "district": [
        "in Reinickendorf", "in Lichtenberg", "in Marzahn-Hellersdorf", "in Treptow-Köpenick",
        "in Tempelhof-Schöneberg", "in Steglitz-Zehlendorf", "in Spandau", "in Kreuzberg", "in Mitte",
        "in Neukölln", "in Charlottenburg", "in Friedrichshain", "in Prenzlauer Berg",
        "in Friedrichshain-Kreuzberg", "in Wedding", "in Pankow", "in Charlottenburg-Wilmersdorf"
    ],
    "pet_friendly": ["pets allowed","pets permitted", "dog friendly", "cat friendly", "animal-friendly","pet friendly"..],
    "floor_level": ["ground floor", "first floor", "second floor", "third floor", "fourth floor", "fifth floor", "top floor", "max 3rd floor", "high floor"..]
}
```

### 🔑 Importance of Paramter Templates:
- These are embedded with SBERT and matched via cosine similarity.
- The better the examples, the more accurate the parsing.
- Enables dynamic, similarity-based mapping rather than relying solely on exact keywords.
- Supports continuous improvement by adding more diverse phrasing.
- Powers both SBERT and fallback regex matching mechanisms.

---

## 📊 Schema and Algorithm Logic for NLP Engine (Mermaid Flowchart)

![SBERT Schema Flowchart](SBERT_FlowChart.png)

## 🧭 SBERT-Based Query Extraction Flow

This flowchart outlines the semantic parameter extraction process for natural-language housing queries using SBERT + Fallback Matching.


| **Step** | **Description** | **Example** |
|----------|------------------|-------------|
| **1. User Query Input** | User types a free-text housing query. | `"Looking for a 3-room furnished apartment near a park under 1000 euros"` |
| **2. Expand Query with Synonyms** | Adds known synonyms (from `synonym_map`) to improve matching. | Adds `"furniture quiet green"` |
| **3. Encode Query using SBERT** | Transforms the full query into a semantic embedding. | SBERT vector of entire query |
| **4. Compute Cosine Similarity** | Compares query vector to template phrase embeddings. | Matches `"3-room apartment"`, `"under 1000"` |
| **5. Similarity ≥ Threshold?** | If match score is high enough, it proceeds. Otherwise fallback kicks in. | ✔ Matches `"rent under 1000"` |
| **6. Extract Parameter & Phrase** | Identifies the parameter (e.g., rent) and the matched phrase. | `"3-room apartment"` → `room_requirement` |
| **7. Check for Value with Regex** | Extract numerical values if needed (rent, area, etc.). | `"under 1000"` → `1000`, `<=` |
| **8. Add to Grouped Parameters** | Store the parameter and value with its operator. | `{parameter: "room_requirement", value: 3, operator: ">="}` |
| **9. Already in Grouped?** | If parameter exists, check if multi-value. | Check if `"furnishing"` already exists |
| **10. Multi-Value Param?** | Append if allowed, else merge or overwrite. | `"feature_request"` → `[balcony, garden]` |
| **11. Next Phrase** | Proceed to next matched phrase. | — |
| **12. Run Fallback Matching** | For low similarity cases, do keyword/token-based check. | `"quiet district"` matches `"quiet"` |
| **13. Match Found?** | If token match found, move to canonical normalization. | `"pet is allowed"` → `"pets allowed"` |
| **14. Normalize Canonical Map** | Normalize synonyms to a standard label. | `"fully equipped"` → `"furnished"` |
| **15. Group with Operator** | Store fallback match in grouped dict with `==`. | `parameter: "environment_preference", value: "green"` |
| **16. Post-Processing & Deduplication** | Remove overlapping/conflicting phrases. | Remove `"garden"` if `"green area"` already included |
| **17. Ensure List for Multi-Params** | Force arrays for multi-value fields. | `"pets allowed"` → `[ "pets allowed" ]` |
| **18. Remove Duplicates & Conflicts** | Final cleanup of repeated/contradicting values. | — |
| **19. Output Final JSON** | Final structured output ready for recommendation engine. | See example below |

---

### 🧾 Sample Final Output

```json
[
  { "priority:1","parameter": "room_requirement", "value": 3, "operator": ">=" },
  { "priority:2","parameter": "price_constraint", "value": 1000, "operator": "<=" },
  { "priority:3","parameter": "furnishing", "value": "furnished", "operator": "==" },
  { "priority:4","parameter": "environment_preference", "value": ["green"], "operator": "==" },
  { "priority:5","parameter": "housing_type", "value": "apartment", "operator": "==" }
]
```

---
## **🚧 Limitations**

1. No LLM Integration (Offline Mode)
The current pipeline does not integrate large language models (LLMs) for fallback reasoning when no parameters are detected. This limits the system’s ability to infer user intent from implicit phrases or uncommon wording.

2.	Synonym Coverage is Manual
The synonym map and canonical normalization require manual maintenance. It may not cover all user phrasing variations, especially for niche or uncommon terms.

3.	Threshold Sensitivity in SBERT Matching
The similarity threshold (0.15) for SBERT phrase matching is heuristically chosen and may miss valid intents if user phrasing is vague or noisy.

4.	Limited Multilingual Support
The system is English-focused. Multilingual queries (e.g., German/Spanish) require extra preprocessing or translation logic.

5.	No Active Context Understanding
The system does not track conversation history or clarify vague intents like “something bigger” or “better location.”

6.	No Online Feedback Loop or Learning
The parser is static — it doesn’t learn or improve based on user corrections or feedback.

7.	Not Robust to Typos or Grammar Errors
Since synonym and SBERT logic rely on exact token or semantic matches, spelling mistakes or highly colloquial input can cause extraction failure.

⸻

## **✅ Recommendations for Improvement**

1.	Integrate Local LLM (e.g., llama.cpp)
Use a lightweight local LLM to generate fallback predictions when SBERT fails to match any parameter. This could significantly improve intent recall.

2.	Synonym Expansion Using Embeddings
Auto-expand the synonym map using vector similarity clustering rather than static lists.

3.	T5 or BART for End-to-End Parsing
Train or fine-tune a T5 model to directly convert user input into structured JSON parameters — especially effective for noisy or complex queries.

4.	Multilingual Support via Preprocessing
Add optional translation preprocessing using MarianMT or similar, for German, Spanish, etc.

5.	Interactive UI for User Feedback
Create a Streamlit/Gradio app to allow users to correct extracted parameters, creating a loop for improvement.

6.	Fuzzy Matching for Typos
Add Levenshtein-distance or fuzzywuzzy-based keyword fallback matching to better handle typos.

⸻

## **🧠 Conclusion**

This SBERT-based NLP engine provides a flexible, modular architecture for converting housing-related queries into structured JSON parameters. It combines semantic similarity, rule-based regex, and fallback logic to ensure robust extraction — even with loosely structured input.

While performant, the pipeline would benefit from LLM integration, learning mechanisms, and richer multilingual/tolerant parsing. With these additions, it can be scaled into a production-grade intent extraction service suitable for real estate platforms or conversational assistants.

---

## 🧩 Task Splitting Suggestions

| Component | Owner Suggestion | Notes |
|----------|------------------|-------|
| Parameter Templates | Maintain training phrases | Analyst / Researcher |
| Synonym Mapping & Canonical Normalization | NLP Engineer | Requires domain-specific input |
| SBERT Template & Embedding Setup | ML Engineer | Model tuning, phrase diversity |
| Regex-based Value Extraction | Price, room, floor, sqm | Backend Developer | Needs constant validation & unit tests |
| Fallback Matching Logic | Regex-based phrase match | NLP Engineer | Improves robustness & recall |
| Post-Processing, Deduplication | Backend Developer | List formatting, canonical checks |
| Final JSON Formatter & Output Structuring | Priority sorting & export| Full Stack | Exposes to API or front-end |
| Testing / Evaluation Setup | QA + ML | Precision/recall metrics per param |

---

## 🧪 Testing Ideas

- ✅ Query coverage: all parameters triggered at least once
- ⚠️ Test overlapping synonyms and value collisions
- 🚫 Test malformed/ambiguous queries
- 📉 Evaluate performance threshold for SBERT matching (default 0.15)

---
