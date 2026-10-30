# Lokaler OCR-Fallback

Interner HTTP-Dienst mit Tesseract (deu+eng) und Poppler. Kein öffentlicher Port,
kein externer OCR-Anbieter, keine Tokenkosten für OCR. Der bereits vorhandene
Groq-Aufruf erhält den erkannten Text anschließend wie bisher.

`docker compose -p normdex-ocr -f incoming-invoices/ocr/compose.yaml up -d --build`
(starten vom Repositoryroot). Restart `unless-stopped`; Docker muss beim Boot starten.
Ein Container, ein aktives Dokument, CPU 0,75, RAM/Swap insgesamt maximal512MiB,
48 Prozesse, kein Root, schreibgeschützte Rootpartition, tmpfs96MiB.
Temporäre Dateien werden je Anfrage im finally-Kontext gelöscht; bei Containerneustart
verschwindet das tmpfs ebenfalls. Keine Dokumentinhalte im HTTP-Log.
Das Image enthält Betriebssystempakete: regelmäßig kontrolliert neu bauen/updaten;
Basisimage-Digest bewusst im Dockerfile aktualisieren.

`POST /ocr`: rohe PDF-Bytes, maximal10MiB. PDFinfo prüft vollständige Seitenzahl;
maximal8Seiten, keine verschlüsselten Dokumente. Seiten einzeln in Graustufen auf
maximal2400Pixel Kantenlänge rendern. Tesseract mit genau einem Thread.
120Sekunden pro Dokument einschließlich Rendering; maximal18.000Zeichen.
Keine Teilverarbeitung großer PDFs: über8Seiten führt zur manuellen Prüfung.
Erfolgreicher Text wird als JSON zurückgegeben, Original-PDF unverändert archiviert.
Unlesbar/ungültig/Limit -> HTTP200 `status:review` -> bestehende Telegram-Prüfmeldung.
Dienstfehler/gleichzeitiger Auftrag -> HTTP503 -> begrenzte n8n-Wiederholung/Queue.

Der n8n-Fallback wird nur bei Extraktionsfehler oder unter40Nicht-Leerzeichen aktiviert.
PDFs mit ausreichendem Text umgehen OCR. Misch-PDFs mit genügend extrahierbarem Text
werden nicht zusätzlich seitenweise OCR-geprüft. Kein automatisches Drehen/Entzerren:
schlechte Scans können weiterhin eine manuelle Prüfung brauchen.

`GET /health` prüft Erreichbarkeit; Docker jede60s, drei Fehlversuche.
`GET /selftest` erkennt eine eingebettete künstliche Scanrechnung mit Marker OCRTEST.
Der tägliche n8n-Gesundheitstest ruft diesen echten OCR-Test auf (still bei Erfolg).
Der stündliche Hostwächter prüft zusätzlich Containerzustand; 120sStartfrist verhindert
Neustart-Fehlalarme. Bei VPS-Ausfall ist auch dieser lokale Wächter betroffen.

Tests ohne echte Dokumente:
- `docker exec -i -e PYTHONPATH=/app normdex-ocr python - < incoming-invoices/ocr/test_service.py`
- `docker cp normdex-ocr:/app/selftest.pdf incoming-invoices/ocr/selftest.pdf`
- `python incoming-invoices/ocr/build_test.py` erzeugt inaktive n8n-Testquellen;
  Testworker vorübergehend publishen, `normdexIncomingOCRTest` im CLI ausführen.
  Dieser Worker umgeht den bereits separat geprüften Datentabellen-Schritt und
  erlaubt ausschließlich dryRun; produktiver Workflow verwendet weiter das Register.
  Nachher Testworker deaktivieren. Keine Testwebhooks oder echten Archivuploads.

Verifiziert 28.10.2026: Scanrechnung->OCR->echteGroq-Inferenz->dry-run,
Textrechnung->direkterPfad, leerePDF->review. Grenztests für Bytes, Seiten,
Verschlüsselung, Timeout und Bereinigung. Acht saubere synthetische Seiten:3,62s;
Containerhöchststand80,9MiB. Keine Zusage für verrauschte/komplexe Scans.
OCR-Container und n8n kontrolliert neu gestartet; aktive Versionen und täglichen
Verbindungstest erfolgreich geprüft. Ein Startphasen-Fehlalarm wurde dabei erkannt,
durch Startfrist korrigiert und aufgelöst.

Rollback: private alte Attachment-/Health-Exports unter
`/home/deploy/workspaces/normdex-ocr/rollback` (0700/0600).
Alte beiden Versionen importieren/publishen und n8n kontrolliert neu starten;
OCR-Containerprüfung im Hostwächter dann entsprechend zurücknehmen.
