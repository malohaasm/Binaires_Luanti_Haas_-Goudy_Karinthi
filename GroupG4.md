# Journal de bord — Service Boutique (G4)

### 09h20 — Vérification du service
- Test du endpoint GET /health en local.
- Résultat : HTTP 200 OK — le service répond correctement.

### 09h30 — Contrat inter-service avec G3
- Confirmation de l’interface exacte de POST /debiter avec l’équipe G3.
- Points validés :
    - Format JSON attendu : `{ "pseudo": "...", "montant": <int> }`

### 09h37 — Début Phase 1.1 : Modèle de données (db.py)
Création des deux tables nécessaires.

```python
class Objet(Base):
    __tablename__ = "objets"
    id: Mapped[int] = mapped_column(primary_key=True)
    nom: Mapped[str]
    prix: Mapped[int]
    item: Mapped[str]  # Exemple : "default:pick_steel 1"

class Livraison(Base):
    __tablename__ = "livraisons"
    id: Mapped[int] = mapped_column(primary_key=True)
    type: Mapped[str] = mapped_column(default="livrer_objet")
    cible: Mapped[str]  # Le pseudo du joueur qui doit recevoir l'objet
    objet: Mapped[str]  # L'itemstring Luanti à livrer
    statut: Mapped[str] = mapped_column(default="en_attente")  # "en_attente" ou "fait"
```

### 09h55 — Auth.py
- Implémentation et validation de la logique JWT.

- Vérification des rôles et du décorateur require_role.


  [Voir le fichier auth.py](./starter/service-boutique/auth.py)

### 09h58 — Catalogue dans app.py
- Début de l’implémentation des routes GET /objets et POST /objets.

- Structure conforme au contrat G4.


  [Voir le fichier app.py](./starter/service-boutique/app.py)

### 10h05 — App.py : POST /acheter terminé
- Implémentation complète de la route POST /acheter.
- Vérification de l’existence de l’objet (404 si introuvable).
- Appel inter-service vers /debiter avec gestion des cas :
    - 503 si le service économie est injoignable.
    - 409 si solde insuffisant.
    - 200 → création d’une livraison en statut "en_attente".
- Insertion en base de la livraison.
- Retourne 201 en cas de succès.
- Tests manuels effectués : OK.

[Voir le fichier app.py](./starter/service-boutique/app.py)

### 10h25 - Rajout Test Auth.py
[Voir le fichier test_auth.py](./starter/service-boutique/test_auth.py)

### 10h40 - Rajout Gestion de Livraison app.py

[Voir le fichier app.py](./starter/service-boutique/app.py)

### 10h48 - Rajout Inventaire app.py

[Voir le fichier app.py](./starter/service-boutique/app.py)

### 10h40 - Partie du Docker-Compose Transmit G1

##  TEST  11h - 12h

### 1. Lister les objets (sans token) → 200
```
Invoke-RestMethod -Uri "http://localhost:8000/objets"
```
### 2. Ajouter sans token → 401
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/objets" `
  -ContentType "application/json" `
-Body '{"nom":"Pioche en fer","prix":10,"item":"default:pick_iron 1"}'
```
### 3. Ajouter avec token joueur → 403
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/objets" `
  -Headers $H_JOUEUR -ContentType "application/json" `
-Body '{"nom":"Pioche en fer","prix":10,"item":"default:pick_iron 1"}'
```
### 4. Ajouter avec token admin → 201
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/objets" `
  -Headers $H_ADMIN -ContentType "application/json" `
-Body '{"nom":"Pioche en fer","prix":10,"item":"default:pick_iron 1"}'
```
### 5. Champ manquant → 400
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/objets" `
  -Headers $H_ADMIN -ContentType "application/json" `
-Body '{"nom":"Pioche en fer"}'
```
### 6. Créditer maxime côté éco d'abord
```
Invoke-RestMethod -Method POST -Uri "http://localhost:5000/crediter" `
  -Headers $H_ADMIN -ContentType "application/json" `
-Body '{"pseudo":"maxime","montant":100}'
```
### 7. Acheter avec solde suffisant → 201
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/acheter" `
  -Headers $H_JOUEUR -ContentType "application/json" `
-Body '{"objet_id":1}'
```
### 8. Acheter sans pièces (après avoir tout dépensé) → 409
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/acheter" `
  -Headers $H_JOUEUR -ContentType "application/json" `
-Body '{"objet_id":1}'
```
### 9. Acheter objet inexistant → 404
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/acheter" `
  -Headers $H_JOUEUR -ContentType "application/json" `
-Body '{"objet_id":9999}'
```
### 10. Acheter sans token → 401
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/acheter" `
  -ContentType "application/json" `
-Body '{"objet_id":1}'
```
### 11. Acheter quand économie est éteinte → 503
#### (Arrêter service-economie avant)
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/acheter" `
  -Headers $H_JOUEUR -ContentType "application/json" `
-Body '{"objet_id":1}'
```
### 12. Lister les livraisons en attente → 200 + liste
```
Invoke-RestMethod -Uri "http://localhost:8000/livraisons" `
-Headers $H_ADMIN
```
### 13. Acquitter une livraison → 200
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/livraisons/1/fait" `
-Headers $H_ADMIN
```
### 14. Acquitter livraison inexistante → 404
```
Invoke-RestMethod -Method POST -Uri "http://localhost:8000/livraisons/9999/fait" `
-Headers $H_ADMIN
```
### 8. Consulter les livraisons en attente
``` 
curl -X GET http://localhost:8080/boutique/livraisons
```
### 9. Acquitter une livraison
```
curl -X POST http://localhost:8080/boutique/livraisons/1/fait
```