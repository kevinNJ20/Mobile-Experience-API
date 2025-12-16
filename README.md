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

Cette API fait des appels HTTP vers:
- **Customers Process API** (port 8082)
- **Bank Accounts Process API** (port 8082)

Configurer dans `global.xml`:
- `Customers_Process_API_Config`
- `Bank_Accounts_Process_API_Config`

### Port

- **Port HTTP**: 8083

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

```bash
curl -X GET "http://localhost:8083/api/mobile/customers/550e8400-e29b-41d4-a716-446655440000"
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

