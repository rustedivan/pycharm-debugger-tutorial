## Snabbintro till PyCharms debugger
...för ska nåt bli gjort får man tydligen göra det själv. ^^

Mkay, så vi snurrar igång med ett enkelt (och vrål-felaktigt) program som räknar fakultet, inbankat i PyCharm. Se nedan.

### Starta debuggern
![Fakultet i Python](/images/1-fib-program.png)

Tryck igång debuggern med knappen i övre högra hörnet, alternativt Ctrl/Cmd-D.

![Debuggerknappen](/images/2-dbg-button.png)

Programmet drar igång som vanligt. (Om inte frågan syns, växla till Console-tabben i nederkanten på förstret, ibland händer det inte automagiskt som det ska). Mata in 5 eller nåt, tryck enter, och titta på när skiten kraschar.
![5 som String](/images/3-int-is-str.png)

### Bugg 1: Programmet kraschar

Alright, felmeddelandet säger nåt om "cannot multiply sequence by non-integer type", vilket inte är direkt lättläst. Växla över till Debugger-tabben istället (nedre delen av fönstret), så borde du se debuggern (som på bilden ovan). This where the trolleri händer.

Längst till vänster finns som vanligt Run/Pause/Stop-knapparna. Till höger om det: Frames-listan, som helt enkelt är en lista över vilka funktioner som anropat varandra för att komma hit. Just nu står vi kraschade i "factorial"-funktionen i main.py:8, som anropats av *&lt;module&gt;*, det vill säga filen själv. (Frames-listan är kanske inte superspännande i ett så här litet program, men i ett större program där 10-15 nästlade funktionsanrop in och ut ur 30-40 filer är vanligt, så är det rätt soft att kunna se hur man hamnade där. Detta kallas för "the call stack".)

I mitten finns Variables, och det är här godiset finns. Alla variabler, deras typer, \_\_exception\_\_ som innehåller all data om vilket fel som inträffat (pröva att fälla ut exception och gräva runt i inälvorna!), och deras värden. Just här ser vi att både **n** och **result** är 5, och det är väl bra... men *typen är {str}*.

Ah, förstås. **raw_input** tar inmatningen som en sträng, det vill säga "5", inte *5*. Svårt att multiplicera text. (Pröva att klicka på *&lt;module&gt;* i callstacken, och kolla att **n** är en sträng där också. Fäll ut **factorial**-funktionen också och kolla runt lite i hur ett funktionsobjekt ser ut, om det är intressant.

*Fix 1: konvertera till integer*

Stoppa programmet. Konvertera resultatet från **raw_input** till integer: *n = int(raw_input("Enter value:"))*

###Bugg 2: MEN DÄ' HÄNDER INGE' (mvh Fucking Åmål)
Dra igång debuggern igen, mata in ett tal, tryck Enter, och notera att inget händer. Programmet har inte kraschat och inte kört klart, så det är väl fortfarande igång, då? Pausa programmet och växla över till Debugger-tabben:

![Inget händer](/images/4-nothing-happens.png)

Det kan ta ett tag innan debuggern kommer igång, pga att vi har torterat datorn ganska rejält, och det är en hel del arbete för debuggern att ens hämta sig från den här missen. Efter ett litet tag borde följande dyka upp:

![Öööh...](/images/5-thats-not-right.png)

Vi kan dra slutsatsen att 5! troligen ska vara mindre än det där resultatet, eller hur? Så hur hamnade vi här? Mja, det enklaste är väl att stega fram koden rad för rad och se hur den beter sig egentligen. 

![Stegkontrollerna](/images/6-step-over.png)

Ovanför variabel-förstret finns de finaste knapparna i hela universum: Step Over, Step Into, Step Out och Run to Cursor. Tryck några gånger på Step Over (vilket helt enkelt innebär: gå till nästa rad) och kolla hur programmet beter sig egentligen. Några gissningar om vad som pågår?

*Förresten:*
- Step Over: gå till nästa rad
- Step Into: gå in i funktionsanrop (om du står vid ett anrop)
- Step Out: gå ut ur funktionen (kör till slutet på funktionen och går ner ett steg i callstacken)
- Run to Cursor: klicka nånstans, tryck till, så kör programmet tills det kommer till den raden. Bäst.

Well, result har blivit larvigt stort eftersom loopen aldrig bryter. Den har stått och multiplicerat tal ganska länge innan vi drog i bromsen. Varför fastnar den? Jo, villkoret är att loopen kör så länge **n >= 0**, men **n** räknar inte ner. Jag har glömt att dra av *1* från *n* i loopen. Sigh.

*Fix 2: räkna ner n*

Stoppa programmet, och lägg in *n -= 1* sist i loopen. Kör igen, och konstatera att programmet kör klart, men...

### Bugg 3: Svaret blir alltid 0

![Det blev ingen CD](/images/7-fib-is-zero.png)

...resultatet blir noll. Spelar ingen roll vad som matas in, 45! = 0. Och innan du ens hinner trycka på Paus är programmet färdigt. Vad göra?

Mjo, vi kan sätta en *breakpoint* i programmet, och be PyCharm att pausa just där när vi kommer dit. Klicka i det grå spåret till vänster om koden, på raden där loopen börjar. Snurra igång debuggern igen, så pausar programmet där. Klicka på Step Over (eller F7) för att stega koden en bit i taget, och försök lista ut vad som går snett.

Ajuste... tänkte inte på det. Till slut kommer ju **n** bli noll, och **result** multipliceras ju med **n** varje varv. Klart vi inte ska multiplicera med 0.

![n = 0](/images/9-n-is-zero.png)

*Fix 3: stoppa loopen tidigare*

![Fixad loop](/images/10-loop-exit.png)

Enkel fix: villkoret i loopen är fett med fel, så stoppa programmet, ändra det till *n > 0* bara, och försök igen. Behåll breakpointen och steppa igenom loopen, och kolla att programpekaren (raden som körs nu) hoppar ur loopen så fort *n == 0*. Ding dong, bugg \#3 är död.

![Kör klart](/images/11-continue-and-read.png)

Ta bort breakpointen, klicka på Continue/Resume-knappen, och växla över till Console-tabben för att kolla resultatet. *Menneh...*

### Bugg 4: 5! är fan inte 600...

Jahapp, nu har vi ett program som inte kraschar, som kör klart, som inte resulterar i noll, men... resultatet är jättefel.