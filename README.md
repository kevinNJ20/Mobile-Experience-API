# Mobile Experience API

API Experience optimisée pour les applications mobiles, fournissant une vue financière agrégée et simplifiée.

## Description

Cette API fournit une vue financière optimisée pour les applications mobiles. Elle agrège les données client et comptes depuis plusieurs Process APIs pour offrir une expérience utilisateur optimale.

## Endpoints

### GET /api/mobile/customers/{customerGlobalId}
Récupère le résumé financier mobile d'un client (compte + comptes + cartes).

**Response:** FinancialSummary object avec:
- Informations client (nom, ID)
- Liste des comptes (numéro, type, solde, devise)
- Liste des cartes (numéro, type, crédit disponible, limite)
- Soldes totaux

## Configuration

### Connexions HTTP Requises

Cette API appelle en parallèle (scatter-gather) :
- **Customers Process API** — `GET /api/customers/{customerGlobalId}`
- **Bank Accounts Process API** — `GET /api/accounts?customerGlobalId=...`

Configurer dans `src/main/resources/config.properties` (ou via propriétés CloudHub) :
- `customers.process.api.host`, `customers.process.api.port`
- `bank.accounts.process.api.host`, `bank.accounts.process.api.port`

### Port et URLs

- **Port HTTP Local** : 8094 (configurable via `http.port`)
- **URL CloudHub** : `https://mobile-experience-api-ng6jme.9d3fd9-2.can-c1.cloudhub.io`

### Déploiement CloudHub (important)

En CloudHub, l’API **ne doit pas** appeler `localhost` : les Process APIs tournent sur d’autres instances. Si les logs CloudHub montrent **Connection refused: localhost/127.0.0.1:8088** ou **:8089**, les propriétés d’application surchargent le fichier embarqué.

**À faire dans Anypoint Runtime Manager** : ouvrir l’application **mobile-experience-api** → **Properties** et définir (avec vos URLs CloudHub réelles) :

| Property | Valeur (exemple CloudHub) | À ne pas utiliser |
|----------|---------------------------|-------------------|
| `customers.process.api.host` | `customers-process-api-ng6jme.9d3fd9-2.can-c1.cloudhub.io` | ~~localhost~~ |
| `customers.process.api.port` | `443` | ~~8089~~ |
| `bank.accounts.process.api.host` | `bank-accounts-process-api-ng6jme.9d3fd9-1.can-c1.cloudhub.io` | ~~localhost~~ |
| `bank.accounts.process.api.port` | `443` | ~~8088~~ |

Le `config.properties` du repo contient déjà ces URLs CloudHub. Si vous avez des propriétés **localhost** définies dans CloudHub, supprimez-les ou remplacez-les par les valeurs ci-dessus, puis redéployez ou redémarrez l’application.

## Architecture Technique

### Flows Business-Logic

- `get-mobile-financial-summary-business-logic`: Agrégation parallèle des données client et financières

### Stratégie d'Agrégation

Le flow utilise un **parallel** pour récupérer simultanément:
1. Informations client depuis Customers Process API
2. Résumé financier (comptes + cartes) depuis Bank Accounts Process API

Puis transforme les données pour un format optimisé mobile (champs simplifiés, calculs de totaux, etc.).

## Exemples de Requêtes

### GET /api/mobile/customers/{customerGlobalId}

**Local:**
```bash
curl -X GET "http://localhost:8094/api/customers/550e8400-e29b-41d4-a716-446655440000"
```

**CloudHub:**
```bash
curl -X GET "https://mobile-experience-api-ng6jme.9d3fd9-2.can-c1.cloudhub.io/api/customers/550e8400-e29b-41d4-a716-446655440000"
```

**Response:**
```json
{
  "customer": {
    "globalId": "550e8400-e29b-41d4-a716-446655440000",
    "customerNumber": "CUST001",
    "name": "John Doe"
  },
  "totalBalance": 5000.00,
  "accounts": [{
    "globalId": "...",
    "accountNumber": "ACC001",
    "accountType": "Checking",
    "accountName": "Compte Courant Principal",
    "balance": 5000.00,
    "availableBalance": 5000.00,
    "currency": "USD"
  }],
  "creditCards": [{
    "globalId": "...",
    "cardNumber": "4111****1111",
    "cardType": "Credit",
    "availableCredit": 4500.00,
    "creditLimit": 5000.00
  }]
}
```

