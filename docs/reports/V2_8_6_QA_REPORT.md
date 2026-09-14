# TEF Vocab Loop v2.8.6 — Calendar Alignment QA

- PASS — JavaScript syntax
- PASS — 2,866 cards retained
- PASS — calendar uses isToday class
- PASS — calendar no longer uses generic today class
- PASS — today styling scoped to calendar
- PASS — home .today layout remains intact

Root-cause check:
- Home `.today` layout CSS remains unchanged.
- Calendar current-day state is now `.calendarDay.isToday`.
- The current date no longer inherits Home margin/grid rules.
