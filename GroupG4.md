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

### 1. Voir le catalogue d'objets
```
curl -X GET http://localhost:8080/boutique/objets
```

### 2. Ajouter un objet — Sans Token
```
curl -X POST http://localhost:8080/boutique/objets \
-H "Content-Type: application/json" \
-d '{"nom": "Arc magique", "prix": 30, "item": "default:bow 1"}'
```

### 3. Ajouter un objet — Joueur normal
```
curl -X POST http://localhost:8080/boutique/objets \
-H "Authorization: Bearer $TOKEN_JOUEUR" \
-H "Content-Type: application/json" \
-d '{"nom": "Arc magique", "prix": 30, "item": "default:bow 1"}'
```

### 4. Ajouter un objet — Admin
```
curl -X POST http://localhost:8080/boutique/objets \
-H "Authorization: Bearer $TOKEN_ADMIN" \
-H "Content-Type: application/json" \
-d '{"nom": "Arc magique", "prix": 30, "item": "default:bow 1"}'
```

### 5. Acheter un objet (Solde suffisant)
```
curl -X POST http://localhost:8080/boutique/acheter \
-H "Authorization: Bearer $TOKEN_JOUEUR" \
-H "Content-Type: application/json" \
-d '{"objet_id": 1}'
```

### 6. Acheter un objet (Solde insuffisant)
```
curl -X POST http://localhost:8080/boutique/acheter \
-H "Authorization: Bearer $TOKEN_JOUEUR" \
-H "Content-Type: application/json" \
-d '{"objet_id": 5}'
```

### 7. Acheter un objet (Économie en panne)
#### (éteindre service-economie avant)
```
curl -X POST http://localhost:8080/boutique/acheter \
-H "Authorization: Bearer $TOKEN_JOUEUR" \
-H "Content-Type: application/json" \
-d '{"objet_id": 1}' 
```


### 8. Consulter les livraisons en attente
``` 
curl -X GET http://localhost:8080/boutique/livraisons
```
### 9. Acquitter une livraison
```
curl -X POST http://localhost:8080/boutique/livraisons/1/fait
```