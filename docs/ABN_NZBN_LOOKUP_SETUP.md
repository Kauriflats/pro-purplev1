# ABN/NZBN Business Lookup Configuration Guide

This document explains how to configure the Australian Business Number (ABN) and New Zealand Business Number (NZBN) lookup services for ListHouze.

## Overview

The ABN and NZBN lookup services allow agencies to verify their business registration during the onboarding process. These services connect to official government APIs to retrieve and validate business information.

## Root Cause Analysis

### Issue
The ABN/NZBN lookup functionality was not working because the required API keys were not configured in Supabase Edge Function secrets.

### Technical Details

| Service | Edge Function | Required Secret | API Provider |
|---------|--------------|-----------------|--------------|
| ABN Lookup | `abn-lookup` | `ABR_GUID` | Australian Business Register (ABR) |
| NZBN Lookup | `nzbn-lookup` | `NZBN_API_KEY` | NZ Ministry of Business, Innovation & Employment (MBIE) |

### Error Flow
1. User enters ABN/NZBN in agency onboarding
2. Frontend calls Supabase Edge Function
3. Edge Function checks for API key in environment variables
4. If key is missing → Returns mock data (development) or error (production)
5. If key is present → Calls external API and returns real data

---

## Configuration Instructions

### 1. Australian Business Number (ABN) Lookup

#### Step 1: Register for ABR Web Services
1. Go to: https://abr.business.gov.au/Tools/WebServices
2. Click "Register for web services"
3. Fill in your organization details
4. Accept the terms and conditions
5. You will receive a **GUID** (authentication key) via email

#### Step 2: Add ABR_GUID to Supabase Secrets
1. Go to your Supabase project dashboard
2. Navigate to **Settings** → **Edge Functions**
3. Click **Manage Secrets**
4. Add a new secret:
   - **Name**: `ABR_GUID`
   - **Value**: Your GUID from ABR registration

#### ABR API Details
- **Endpoint**: `https://abr.business.gov.au/abrxmlsearch/AbrXmlSearch.asmx/ABRSearchByABN`
- **Documentation**: https://abr.business.gov.au/Documentation/WebServiceResponse
- **Rate Limit**: No official limit, but be reasonable (< 100 requests/minute)
- **Cost**: Free

---

### 2. New Zealand Business Number (NZBN) Lookup

#### Step 1: Register for MBIE API Access
1. Go to: https://api.business.govt.nz/
2. Create an account or sign in
3. Navigate to **API Products** → **NZBN v5**
4. Subscribe to the API
5. Generate API credentials (Bearer token)

#### Step 2: Add NZBN_API_KEY to Supabase Secrets
1. Go to your Supabase project dashboard
2. Navigate to **Settings** → **Edge Functions**
3. Click **Manage Secrets**
4. Add a new secret:
   - **Name**: `NZBN_API_KEY`
   - **Value**: Your Bearer token from MBIE

#### MBIE API Details
- **Endpoint**: `https://api.business.govt.nz/gateway/nzbn/v5/entities/{nzbn}`
- **Documentation**: https://api.business.govt.nz/api/explore-apis/nzbn
- **Rate Limit**: Varies by subscription tier
- **Cost**: Free tier available with limited calls

---

## Development Mode (Mock Data)

When the API keys are not configured, the Edge Functions now return **mock data** instead of errors. This allows development and testing without requiring real API credentials.

### Mock Data Response Format

**ABN Mock Response:**
```json
{
  "success": true,
  "result": {
    "abn": "12345678901",
    "businessName": "Test Business 1234",
    "entityType": "Company",
    "state": "VIC",
    "postcode": "3000",
    "isActive": true
  },
  "mock": true,
  "message": "Using mock data. Configure ABR_GUID in Supabase Edge Function secrets for real ABN lookups."
}
```

**NZBN Mock Response:**
```json
{
  "nzbn": "1234567890123",
  "entityName": "Test NZ Business 1234",
  "entityType": "NZ Limited Company",
  "entityStatus": "Registered",
  "registrationDate": "2020-01-15",
  "address": {
    "address1": "123 Queen Street",
    "city": "Auckland",
    "postCode": "1010",
    "country": "New Zealand"
  },
  "isMock": true,
  "message": "Using mock data. Configure NZBN_API_KEY in Supabase Edge Function secrets for real NZBN lookups."
}
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "ABN lookup service is not configured" | `ABR_GUID` not set | Add ABR_GUID to Supabase secrets |
| "NZBN lookup service is not configured" | `NZBN_API_KEY` not set | Add NZBN_API_KEY to Supabase secrets |
| "ABN verification service authentication failed" | Invalid or expired GUID | Re-register at ABR and get new GUID |
| "NZBN API authentication failed" | Invalid or expired token | Regenerate API token at MBIE |
| "ABN not found" | ABN doesn't exist or is cancelled | Verify ABN is correct and active |
| "NZBN not found" | NZBN doesn't exist | Verify NZBN is correct |
| "Rate limit exceeded" | Too many requests | Wait 1 minute and retry |

### Checking Edge Function Logs

To debug issues, check the Edge Function logs:

1. Go to Supabase Dashboard
2. Navigate to **Edge Functions**
3. Select `abn-lookup` or `nzbn-lookup`
4. Click **Logs**
5. Look for entries like:
   - `[ABN Lookup] ABR_GUID configured: YES/NO`
   - `[ABN Lookup] Calling ABR API...`
   - `[ABN Lookup] ABR API returned status: XXX`

---

## Security Considerations

1. **Never expose API keys** in frontend code
2. API keys should only be stored in Supabase Edge Function secrets
3. The Edge Functions act as a proxy to protect credentials
4. Rate limiting is implemented on NZBN lookups (30/minute)

---

## Files Modified

| File | Purpose |
|------|---------|
| `supabase/functions/abn-lookup/index.ts` | ABN lookup Edge Function |
| `supabase/functions/nzbn-lookup/index.ts` | NZBN lookup Edge Function |
| `src/components/onboarding/ABNLookup.tsx` | Frontend ABN lookup component |
| `src/pages/onboarding/AgencyOnboarding.tsx` | Agency onboarding with lookup |

---

## Testing

### Test ABN Numbers (Australia)
- Valid format: 11 digits (e.g., `51824753556`)
- ABR provides test ABNs for development

### Test NZBN Numbers (New Zealand)
- Valid format: 13 digits (e.g., `9429041537077`)
- Use mock mode for development

---

## Contact & Support

For API registration issues:
- **ABR**: https://abr.business.gov.au/Contact
- **MBIE**: https://api.business.govt.nz/support

For ListHouze platform issues:
- Create an issue at the project repository
