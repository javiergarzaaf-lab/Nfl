# Ejemplo: Análisis Completo de un Partido Usando los 3 Skills Juntos

## Escenario: Analizar a fondo el matchup **Kansas City Chiefs vs Las Vegas Raiders**

### 1️⃣ Traer Datos Básicos (skill: `nfl-data`)

```
Claude: "Trae los scores de esta semana y el standings actual de la AFC West"
→ sports-skills nfl get_scoreboard
→ sports-skills nfl get_standings --season=2025
```

**Output:**
- Marcadores en vivo (todas las semanas actuales)
- Ranking de AFC West: 1. KC (x-x), 2. LV (x-x), 3. LAC, 4. DEN
- Proyecciones de playoff

---

### 2️⃣ Scout del Matchup (skill: `nfl-matchup-scout`)

```
Claude: "Genera un reporte de ventajas entre KC (ofensa) y LV (defensa)"
→ python -m nfl_scout KC LV --seasons 2025 2026 > report.md
```

**Output:** Reporte Markdown con todas las brechas:

```markdown
## KC offense — attack points vs LV
1. ⚡ MEDIUM-LEVERAGE EXPLOIT — KC off-tackle run vs LV defense
   - KC: +0.18 EPA/play, 56% success (z +1.9, n=142)
   - LV: -0.22 EPA/play, 38% success (z -1.7, n=89)
   - League baseline: +0.08 EPA/play
   - Freq: ~6.2 times/game for KC historically
   - Expected edge: +1.1 EPA/game

2. 📊 LOW-LEVERAGE CAUTION — KC under center vs LV zone
   - KC: -0.19 EPA/play, 30% success (z -1.9, n=43)
   - LV: -0.07 EPA/play, 46% success (z +1.0, n=79)
   - Freq: ~2.5 times/game for KC

## LV offense — attack points vs KC
(Invertir el análisis: dónde LV es fuerte y KC es débil)

## Tendencias a evitar
(Donde KC/LV es débil y el oponente es fuerte)
```

**Esto explica:**
- ✓ Dónde KC puede atacar (run vs coverage específica)
- ✓ Dónde LV puede atacar
- ✗ Qué debe evitar cada equipo
- → EPA esperado por gameplan

---

### 3️⃣ Scheme Matrix (skill: `nfl-scheme-matrix`)

```
Claude: "Muestra el scheme matrix y los gaps del modelo para KC vs LV"
→ nfl-model best-bets --limit 5 > scheme_report.md
→ nfl-model export > full_scheme.json
```

**Output JSON (reformatted):**

```json
{
  "authority": "RESEARCH_ONLY",
  "top_spreads": [
    {
      "game": "KC @ LV",
      "model": "KC -8.5",
      "market": "KC -7.0",
      "gap": 1.5,
      "reason": "KC uses shotgun 54% | 11 personnel 68% | LV projects 35% man / 65% zone | Blitz 28%, pressure 32%"
    }
  ],
  "teams": {
    "KC": {
      "personnel": {
        "11": { "rate": 0.68, "epa_per_play": 0.24 },
        "12": { "rate": 0.18, "epa_per_play": 0.11 },
        "21": { "rate": 0.10, "epa_per_play": 0.08 }
      },
      "coverage": {
        "man": { "rate": 0.35, "epa_allowed": -0.08 },
        "zone": { "rate": 0.65, "epa_allowed": -0.14 }
      },
      "pressure": {
        "blitz_rate": 0.28,
        "pressure_rate": 0.32
      },
      "formation": {
        "shotgun": 0.54,
        "under_center": 0.38,
        "pistol": 0.08
      }
    },
    "LV": { ... similar structure ... }
  },
  "live_reaction": {
    "KC": {
      "offense_pass_epa_trend": "+0.047",  // Mejorando vs último mes
      "defense_pressure_trend": "-0.023"   // Presionando menos
    }
  }
}
```

**Esto revela:**
- ✓ Personnel tendencias (cuándo cada equipo juega)
- ✓ Coverage preferences (man vs zone)
- ✓ Pressure rates vs effectiveness
- ✓ Cambios vivos (últimas 4 semanas)
- ✓ Cómo esas preferences se alinean/desalinean

---

### 🎯 Síntesis: Reporte Ejecutivo

Con los 3 datos, ahora puedes escribir:

---

## **MATCHUP SCOUT: KC @ LV, Semana X de 2025**

### Standings & Context
- **AFC West:** KC (8-2) vs LV (4-6) — KC aseguró división
- **Línea:** KC -7.0 (mercado) vs KC -8.5 (modelo) — gap de 1.5 puntos

### Exploits Ofensivos de KC vs Defensa de LV

| Esquema | EPA/play (KC) | Success % | Sample | Freq (juegos) | Edge Esperado |
|---------|---------------|-----------|--------|---------------|---------------|
| Off-tackle run vs 5-DB nickel | +0.18 | 56% | n=142 | 6.2 | +1.1 EPA |
| Shotgun 11 personnel vs zone 3 | +0.14 | 52% | n=87 | 5.8 | +0.8 EPA |
| Play action vs man coverage | +0.22 | 58% | n=34 | 2.1 | +0.5 EPA |
| **Recomendación KC:** Focus on outside zone runs, shotgun 11 personnel en zone, PA vs man |

### Exploits Ofensivos de LV vs Defensa de KC

| Esquema | EPA/play (LV) | Success % | Sample | Freq (juegos) | Edge Esperado |
|---------|---|---|---|---|---|
| Inside run vs 7-man box | -0.24 | 31% | n=54 | 3.2 | -0.8 EPA |
| 13 personnel vs nickel | -0.73 | 27% | n=48 | 2.8 | -1.3 EPA |
| 3rd-down pass vs blitz | +0.08 | 45% | n=62 | 4.1 | +0.3 EPA |
| **Recomendación LV:** Evitar inside runs / 13 personnel — atacar 3rd down si KC blitzeea |

### Scheme Tendencias (Matriz Viva)

**Kansas City (Ofensa):**
- Personnel: 68% en 11 (RB + TE), 18% en 12 (2 TE), 10% en 21 (2 RB)
- Formación: 54% Shotgun, 38% under-center, 8% pistol
- Play action: 22% (liga: 18%) — más frequent, +0.09 EPA edge
- EPA by coverage: +0.24 vs man, +0.18 vs zone

**Las Vegas (Defensa):**
- Coverage: 35% man, 65% zone — defensiva orientada a cobertura
- Base/Nickel: 45% base (KC probablemente irá a 11 personnel shotgun)
- Blitz rate: 28% (liga: 32%) — conservadora
- EPA allowed vs shotgun: -0.16, vs under-center: -0.09

**Cambios Vivos (últimas 4 semanas):**
- KC offense: Pass EPA +0.047 trend (calentándose)
- LV defense: Pressure rate -0.023 trend (presionando menos)

### Conclusión

✅ **Alta confianza:** KC debe atacar con corredores (off-tackle vs nickel) y play action.  
⚠️ **Riesgo LV:** Defensiva débil vs formaciones base KC, pero puede defenderse 3rd down.  
📊 **Proyección:** KC probablemente gana ~8-10 puntos; modelo dice -8.5, mercado -7.0.

---

## 📋 Cómo Pedirle a Claude que Haga Esto

```
"Haz un análisis completo del partido KC vs LV. 
Pasos:
1. Usa nfl-data para traer marcador, standings, lesiones
2. Genera un reporte de matchup-scout mostrando donde KC puede atacar
3. Corre nfl-model best-bets para ver el scheme matrix vivo
4. Resume en una tabla: exploits, schema tendencias, conclusión"
```

Cada skill corre automáticamente y produce su output:
- **nfl-data** → scores + standings
- **nfl-matchup-scout** → KC off-tackle vs LV nickel (+1.1 EPA) 
- **nfl-scheme-matrix** → Personnel distros, coverage prefs, trends vivos

**Resultado:** Reporte de 2-3 páginas listo para presentación, pregame, o preparación de análisis.

---

**Skills disponibles en:** `/home/user/Nfl/.claude/skills/`  
**Repos fuente en:** `/home/user/Nfl/nfl-analisis/repos/`
