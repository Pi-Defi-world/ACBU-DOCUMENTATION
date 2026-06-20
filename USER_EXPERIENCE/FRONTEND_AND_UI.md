# Frontend Architecture and UI/UX Guide

## 1. Purpose
This document defines the frontend architecture, design language, and user experience standards for all ACBU products.

## 2. Frontend Technology Stack
- **Framework:** Next.js 15+
- **Language:** TypeScript
- **Styling:** Tailwind CSS, CSS Variables
- **State:** Zustand
- **Forms:** React Hook Form, Zod
- **API:** TanStack Query
- **Wallet:** Stellar Wallet Kit, Freighter
- **Charts:** Recharts
- **Icons:** Lucide React

## 3. Application Architecture
*Application Architecture defines the structural foundation of our products. See Section 6 for Page Structure detailing specific sections.*

## 4. Design Principles
### Design Philosophy
- Simple
- Trustworthy
- Financial-grade
- Mobile-first
- Accessible
- African-focused

### Core Design Rules
- Maximum 3 primary actions per page
- Clear hierarchy
- No hidden balances
- Transparent fees
- Always show:
  - exchange rates
  - fees
  - transaction status

## 5. UI Design System
### Colors
- **Primary:** Emerald / Green
- **Secondary:** Deep Blue
- **Success:** Green
- **Warning:** Amber
- **Error:** Red
- **Neutral:** Gray Scale

### Typography
- **Headings:** Inter Bold
- **Body:** Inter Regular
- **Numbers:** Tabular Numeric

## 6. Page Structure
### Main Pages

#### Landing
- Value proposition
- Reserve transparency
- Basket composition
- CTA

#### Dashboard
- Balance
- Basket Value
- Recent Transactions
- Reserve Health

#### Transfer
- Recipient
- Amount
- Fees
- Preview
- Confirm

#### Mint
- Deposit Currency
- Amount
- Rate
- Preview
- Mint

#### Burn
- Redeem
- Receive Currency
- Settlement ETA

## 7. Component Library
- Button
- Input
- Select
- Card
- Modal
- Toast
- Balance Card
- Reserve Dashboard
- Exchange Widget
- Currency Basket Widget
- Transaction Timeline
- KYC Status Card
- Merchant Settlement Card

## 8. Responsive Design Standards
*Standards for fluid layout, padding, margins, and component scaling across viewport sizes.*

## 9. Accessibility Standards
- WCAG 2.1 AA
- Keyboard navigation
- ARIA labels
- Screen reader support
- 4.5:1 contrast minimum

## 10. User Experience Guidelines
*Focus on clear user feedback, minimal friction, and intuitive financial interactions.*

## 11. Design Tokens
*Reusable values storing our design decisions (colors, typography, spacing).*

## 12. Error Handling UX
*How to gracefully recover and communicate effectively during transaction failures, network issues, or validation errors.*

## 13. Loading States
*Implementation of optimistic UI, skeleton loaders, and asynchronous progress indicators.*

## 14. Mobile Experience
### Mobile Standards
- Bottom navigation
- Large tap targets
- Offline indicators
- Low bandwidth optimization

## 15. Future Mobile App Alignment
*Guidelines ensuring consistent architectural patterns to facilitate future React Native / cross-platform mobile implementations.*
