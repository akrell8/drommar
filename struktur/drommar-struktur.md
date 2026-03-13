# Drömmar – Struktur, metodik och arbetsflöde
Ett sammanhållet system för att utforska, planera och versionera livets stora och små projekt: utbildning, yrken, resor, hobbies och långsiktiga livsdrömmar. Systemet bygger på: GitHub → lagring, versionering, spårbarhet, projektstyrning. Obsidian → utforskning, tankestruktur, länkar, graph view. Separata repon + submodules → modularitet och skalbarhet. Issues/Projects → idéer, planer, research, beslut.

## 1. Övergripande arkitektur
Du använder en root-repo som nav, och flera underrepon som representerar olika livsområden.

### 1.1 Root-repo: `drommar`
Funktion: nav, index, meta, överblick. Innehåll: README.md – syfte, koncept, hur allt hänger ihop. INDEX.md – länkar till alla underrepon. ROADMAP.md – långsiktiga mål och teman. projects/ – GitHub Projects för planering. issues/ – idéer, researchfrågor, tankespår. meta/ – principer, arbetsflöden, konventioner. Detta repo är din “kontrollpanel”.

## 2. Underrepon per livsområde
Varje område får ett eget repo för modularitet och spårbarhet. Exempel: drommar-utbildning, drommar-yrke, drommar-resor, drommar-hobbies, drommar-liv, drommar-psykologi. Varje repo innehåller: README.md – syfte och struktur. notes/ – Obsidian-vänliga markdownfiler. research/ – utdrag, länkar, analyser. plans/ – konkreta planer, steg, beslut. history/ – versionsloggar, milstolpar. issues/ – tankespår, frågor, alternativ. projects/ – lokala projektboards.

## 3. Underrepon i GitHub – hur det fungerar
GitHub har inte stöd för “repon i repon” som mappar. Men du kan göra detta på två sätt:

### 3.1 Separata repon (det vanligaste)
Du skapar flera repon och länkar dem via root-repo-index, GitHub-organisation och Obsidian-struktur. Detta är enkelt, robust och passar perfekt för livsdomäner.

### 3.2 Git submodules (om du vill ha allt i ett träd lokalt)
Submodules låter dig inkludera andra repon i root-repon. Exempel:  
git submodule add https://github.com/tomas/drommar-utbildning utbildning/  
git submodule add https://github.com/tomas/drommar-yrke yrke/  
git submodule add https://github.com/tomas/drommar-resor resor/  
Fördelar: Root-repo blir ett nav med alla moduler. Obsidian kan peka på root och se allt. Varje modul har egen historik och issues. Detta är den mest professionella strukturen.

## 4. Rekommenderad strategi: börja utan submodules, lägg till dem senare
För att få maximal flexibilitet i början och undvika onödig Git-friktion rekommenderas följande strategi:

### 4.1 Börja utan submodules
1. Skapa en lokal struktur:  
Drommar/  
  README.md  
  INDEX.md  
  ROADMAP.md  
  meta/  
  projects/  
  issues/  
  utbildning/  
  yrke/  
  resor/  
  hobbies/  
  liv/  
  psykologi/  
2. Initiera root-repon:  
cd Drommar  
git init  
git add .  
git commit -m "Initial structure"  
3. Push till GitHub som vanligt.  
4. Låt underkatalogerna vara vanliga mappar tills du känner att en domän är mogen att bli ett eget repo.  
5. När du är redo att separera en domän:  
cd Drommar/utbildning  
git init  
git add .  
git commit -m "Initial import"  
git remote add origin <URL till drommar-utbildning>  
git push -u origin main  

Detta ger dig frihet att forma strukturen innan du modulariserar den.

### 4.2 Lägg till submodules när strukturen stabiliserats
När du vill att root-repon ska fungera som ett nav med moduler:
1. Ta bort den lokala mappen (innehållet finns redan i GitHub):  
rm -rf utbildning/  
2. Lägg till submodulen:  
git submodule add <URL till drommar-utbildning> utbildning/  
3. Committa pointern:  
git add .  
git commit -m "Added submodule: utbildning"  
git push  
4. Uppdatera submodules vid behov:  
git submodule update --init --recursive  

Detta ger dig ett professionellt, modulärt och skalbart system.

## 5. Obsidian som frontend
Obsidian pekar på en lokal klon av hela Drömmar-trädet. Rekommenderad struktur:  
/Vault  
  /Drömmar (root-repo)  
    /utbildning (submodule eller mapp)  
    /yrke  
    /resor  
    /hobbies  
    /liv  
    /psykologi  
Fördelar: Backlinks och graph view ger överblick. Dataview kan skapa dashboards. Du kan jobba snabbt och associativt. GitHub tar hand om versionering.

## 6. GitHub Projects + Issues = Spårbarhet
### 6.1 Issues
Används för: idéer, researchfrågor, alternativjämförelser, beslut, “drömmar” som ska konkretiseras.

### 6.2 Projects
Används för: kanban för livsplanering, roadmaps för utbildningsspår, research pipelines, hobbies och experiment. GitHub Projects är repo-agnostiskt, så root-repon kan ha globala boards som inkluderar issues från alla underrepon.

## 7. Konventioner och arbetsflöden
### 7.1 Filnamn
YYYY-MM-DD-titel.md för dagboks-/tankeanteckningar. tema-namn.md för konceptuella dokument. plan-xxx.md för planer.

### 7.2 Taggar (Obsidian)
#dröm, #plan, #research, #utbildning, #psykologi, #yrke, #resa, #hobby, #liv.

### 7.3 Branching
idea/<ämne>, research/<område>, plan/<projekt>, decision/<beslut>.

### 7.4 Releases
Exempel: v1.0-utbildningsplan, v1.0-livsstrategi, v1.0-psykologi-roadmap.

## 8. Koppling till dina aktuella utbildningsspår
Du har två centrala utbildningslinjer just nu:

### 8.1 Psykologprogrammet (PSPRO)
Urval: BI + HP. Dina meriter: BI: 20,00. HP: beroende på prov. Akademiska poäng används inte. Placering: stark i BI, men konkurrensen är extrem.

### 8.2 Psykologi II (PS22KP)
Urval: BI + APAV/AP + HP. Dina meriter: BI: 20,00. APAV/AP: ≈ 315 hp. Placering: mycket stark i både BI och hp-urval.

## 9. Koppling till övriga kurser du nämnt
Kurser: FE9020 / SU-31107, FE9302 / SU-31120, DD2482 / KTH-11829, ME1035 / KTH-10971, ME2088 / KTH-10973, DD100N / KTH-11827, IK2206 / KTH-11845, AS1011 / SU-46029. Urval: BI + HP + APAV/AP/APS. Din position: BI: 20,00 → mycket starkt. APAV/AP/APS: ≈ 315 hp → toppsegmentet. HP: valfritt komplement. Du ligger alltså mycket bra till för alla dessa kurser.

## 10. Samlad slutsats
Du bygger nu ett system som är tekniskt robust, skalbart, ger spårbarhet över tid, kombinerar GitHub och Obsidian på ett elegant sätt och låter dig utforska och planera livet med samma precision som dina tekniska projekt. Dina utbildningsmeriter är: BI: 20,00. Akademiska poäng: ≈ 315 hp. Extremt stark för fristående kurser. Konkurrenskraftig för program via BI/HP. Detta är en solid grund för både struktur och framtida beslut.


