## Snabbintro till PyCharms debugger
...för ska nåt bli gjort får man tydligen göra det själv. ^^

Mkay, så vi snurrar igång med ett enkelt (och vrål-felaktigt) program som räknar Fibonacci-tal, inbankat i PyCharm. Se nedan.

### Starta debuggern
![Fibonacci i Python](/images/1-fib-program.png)

Tryck igång debuggern med knappen i övre högra hörnet, alternativt Ctrl/Cmd-D.

![Debuggerknappen](/images/2-dbg-button.png)

Programmet drar igång som vanligt. (Om inte frågan syns, växla till Console-tabben i nederkanten på förstret, ibland händer det inte automagiskt som det ska). Mata in 5 eller nåt, tryck enter, och titta på när skiten kraschar.
![5 som String](/images/3-int-is-str.png)

### Bugg 1: Programmet kraschar

Alright, felmeddelandet säger nåt om "cannot multiply sequence by non-integer type", vilket inte är direkt lättläst. Växla över till Debugger-tabben istället (nedre delen av fönstret), så borde du se debuggern (som på bilden ovan). This where the trolleri händer.

Längst till vänster finns som vanligt Run/Pause/Stop-knapparna. Till höger om det: Frames-listan, som helt enkelt är en lista över vilka funktioner som anropat varandra för att komma hit. Just nu står vi kraschade i "factorial"-funktionen i main.py:8, som anropats av \<module\>, det vill säga filen själv. (Frames-listan är kanske inte superspännande i ett så här litet program, men i ett större program där 10-15 nästlade funktionsanrop in och ut ur 30-40 filer är vanligt, så är det rätt soft att kunna se hur man hamnade där. Detta kallas för "the call stack".)

I mitten finns Variables, och det är här godiset finns. Alla variabler, deras typer, \_\_exception\_\_ som innehåller all data om vilket fel som inträffat (pröva att fälla ut exception och gräva runt i inälvorna!), och deras värden. Just här ser vi att både **n** och **result** är 5, och det är väl bra... men *typen är {str}*.

Ah, förstås. **raw_input** tar inmatningen som en sträng, det vill säga "5", inte *5*. Svårt att multiplicera text. (Pröva att klicka på \<module\> i callstacken, och kolla att **n** är en sträng där också. Fäll ut **factorial**-funktionen också och kolla runt lite i hur ett funktionsobjekt ser ut, om det är intressant.

*Fix 1: konvertera till integer*

Konvertera resultatet från **raw_input** till integer: *n = int(raw_input("Enter value:"))*
