# VALIDATE Skill

> יועץ ולידציה לתוצרי AI. תופס שגיאות לפני שהן נשלחות.

**Built by [Guy Cohen](https://www.linkedin.com/in/guycohen-ai/) - AI consultant, Israel.**

## מה זה עושה

`VALIDATE` הוא סקיל שמופעל אחרי כל המלצה, תכנון, או ארכיטקטורה שקלוד נותן לך. הוא מריץ סוכן ביקורתי שמדרג כל ממצא לפי חומרה ומחזיר ציון ברור:

- **BLOCKER** - ייכשל אם תשלח כך
- **RISK** - כיוון נכון, מספרים שצריך לכוון
- **PREFERENCE** - הייתי עושה אחרת, אבל הגיוני
- **CORRECTLY-IDENTIFIED** - נקודה שכבר נתפסה היטב

ואז ורדיקט סופי: **PASS / CALIBRATE / REJECT**.

## למה זה דרוש

קלוד נוטה לאופטימיזם שיטתי. הוא נותן המלצות שנשמעות מצוין אבל המספרים מנופחים, הלוחות זמנים אבסורדיים, וההנחות לא בדוקות. `VALIDATE` נלחם בנטייה הזאת על ידי הכרחה של דירוג חומרה במקום ערמת חסמים אקראית.

**הסקיל הזה תפס אצלי 84 פעמים בחודשיים האחרונים שקלוד טעה.**

## התקנה ל-Claude Code כתוסף (מומלץ)

בתוך Claude Code:

```
/plugin marketplace add guycoful/validate-skill
/plugin install validate@validate-skill
```

עדכונים מגיעים אוטומטית מהריפו.

## התקנה ידנית ל-Claude Code

```bash
# מוודא שיש לך תיקיית skills בקלוד
mkdir -p ~/.claude/skills/validate

# מוריד את הסקיל
curl -o ~/.claude/skills/validate/SKILL.md \
  https://raw.githubusercontent.com/guycoful/validate-skill/main/skills/validate/SKILL.md
```

או פשוט להוריד את הקובץ `skills/validate/SKILL.md` ולשים אותו ב:
- **Windows:** `C:\Users\<your-user>\.claude\skills\validate\SKILL.md`
- **macOS/Linux:** `~/.claude/skills/validate/SKILL.md`

אחרי שהקובץ במקום, פתח שיחה חדשה ב-Claude Code. הסקיל יזוהה אוטומטית.

## התקנה ל-Codex

```bash
# מוודא שיש לך תיקיית agents בקודקס
mkdir -p ~/.codex/agents/validate

# מוריד את הקובץ Codex-compatible
curl -o ~/.codex/agents/validate/AGENTS.md \
  https://raw.githubusercontent.com/guycoful/validate-skill/main/AGENTS.md
```

או להוריד את `AGENTS.md` ולשים אותו במיקום המתאים בקודקס שלך.

## שימוש

הסקיל מופעל אוטומטית אחרי כל ריצה של `/advisor`. אפשר גם להפעיל ידנית:

```
/validate <תוכן ההמלצה שרוצים לאמת>
```

## דוגמת ריצה

```
User: /advisor איזו ארכיטקטורה לבוט AI

Advisor: [המלצה עם KNOWN/ASSUMED/MISSING וטווחים]

Validator (אוטומטי):
  - BLOCKER: Railway אין בו free tier
  - RISK: הערכת עלות tokens נמוכה ב-30%
  - CORRECTLY-IDENTIFIED: גישת auth נכונה
  - Verdict: REJECT

→ ניסוח מחדש עם Hetzner במקום Railway
→ ולידציה חוזרת: PASS עם קליברציות
```

## סקילים אחרים שלי

- [negotiate-skill](https://github.com/guycoful/negotiate-skill) - יועץ משא ומתן
- [aeo-skill](https://github.com/guycoful/aeo-skill) - אופטימיזציה למנועי AI
- [codex-skill](https://github.com/guycoful/codex-skill) - תרגום משימה לפרומפט Codex

## רישיון

MIT
