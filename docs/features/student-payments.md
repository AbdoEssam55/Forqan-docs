# Student Payments

## Overview

Student payments track fees and payments for students. Allocations allow splitting a payment across multiple items or students. The API is typically gated by the Payments module. All endpoints require tenant authentication and appropriate tenant claims (e.g. ViewStudentPayments, Create, Update, Delete). Some endpoints may support file upload (e.g. receipt).

## Base route

- **Payments**: `/API/V01/StudentPayments`
- **Allocations**: `/API/V01/StudentPaymentAllocations`

## Main operations (StudentPayments)

- **List**: `POST List` or equivalent with pagination/filters
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`
- **File upload**: If supported, see controller for receipt/attachment endpoint

## Allocations (StudentPaymentAllocations)

- CRUD for payment allocations (linking payments to students or line items). See controller for exact routes.

## Related documentation

- [Security – payment API guidelines](../security/payment-api-guidelines.md)
- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md) – PaymentsBasic or equivalent module

## Controllers

`Alforqan.Api/Controllers/StudentPayments/StudentPaymentsController.cs`  
`Alforqan.Api/Controllers/StudentPaymentAllocations/StudentPaymentAllocationsController.cs`
