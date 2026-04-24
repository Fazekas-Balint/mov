# WebM -> HEVC+alpha MOV konverter (GitHub Actions)

Ez a repo csak arra van, hogy a `videos/` mappában lévő átlátszó WebM videókat
ingyenesen HEVC+alpha MOV formátumra konvertáljuk egy **GitHub-hosted macOS runneren**
(ez az egyetlen gyors, ingyenes út a `hevc_videotoolbox` enkódolóhoz, ami tudja az alphát).

## Miért kell

iOS Safari nem támogatja a WebM alpha channelt -> átlátszó pixelek feketén jelennek meg.
Ha a `<video>` mellé teszünk egy `<source>`-t HEVC+alpha MOV-ra is, iOS az MOV-ot
használja (átlátszó), Chrome/Firefox/Edge a WebM-et (átlátszó marad).

## Lépések (egyszeri setup, kb. 10 perc)

1. **Csinálj egy GitHub repo-t**
   - Bárhol: https://github.com/new
   - Fontos: **Public**-ra állítsd -> macOS runner ingyenes korlátlanul.
     (Private repónál a macOS perc 10x szorzóval megy, free tier 500 perc total/hó,
     azaz ~50 macOS perc. Ehhez a feladathoz bőven elég lenne, de public biztosabb.)
   - Név mindegy, pl. `webm-hevc-convert`.

2. **Push-old fel ezt a mappát**
   ```bash
   cd C:\Users\User\PhpStormProjects\mov
   git init
   git add .
   git commit -m "init"
   git branch -M main
   git remote add origin https://github.com/<YOUR_USER>/<REPO_NAME>.git
   git push -u origin main
   ```
   A push automatikusan elindítja a workflow-t (a `push` trigger miatt).

3. **Várd meg, hogy lefusson**
   - GitHub repo -> **Actions** tab -> `Convert WebM -> HEVC+alpha MOV` workflow
   - Kb. 2-3 perc alatt lefut (a 32MB-os `task_introducing.webm` a legnagyobb)

4. **Töltsd le az artifact-ot**
   - A workflow run oldalán alul: **Artifacts** -> `hevc-alpha-movs` -> letölti zipben
   - Kicsomagolod, 6 db MOV

5. **Másold a MOV-okat a projektbe**
   ```
   C:\Users\User\Documents\develop\szitar-task\web\dist\mp4\
   ```
   Az eredeti WebM-ek maradnak, a MOV-ok melléjük kerülnek.

6. **Frissítsd a HTML-t** -> `<source>` elemekkel mindkét formátumot felajánlod.
   (Ezt külön kérd, megírom az index.php diff-et, ha kész a konverzió.)

## Workflow újrafuttatás

Ha később megváltozik egy WebM vagy újat teszel a `videos/` mappába:
- Push-old a változást -> workflow automatikusan újra lefut.
- Vagy manuálisan: Actions tab -> workflow -> `Run workflow` gomb (ez a `workflow_dispatch` trigger).

## Ellenőrzés (a workflow automatikusan csinálja)

A workflow `ffprobe`-bal minden kimeneti MOV-nál kiírja:
- `codec_name=hevc` (kell)
- `pix_fmt=bgra` vagy hasonló alpha-t tartalmazó formátum (kell - ez jelzi, hogy benne van az alpha)

Ha `pix_fmt=yuv420p` (vagy bármi `a` nélküli), az alpha elveszett -> jelezd, javítjuk az ffmpeg parancsot.
