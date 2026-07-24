---
title: "Qubefini RBAC Deployment Steps"
date: 2026-07-24
tags:
  - qubefini
  - work
  - rbac
  - deployment
publish: false
---

# Qubefini RBAC Deployment Steps

1. First run `npx prisma@6 migrate deploy`
2. Then run `go run ./cmd/seeder/main.go seed-features-permissions` to seed all the features and permissions
3. Then run `go run ./cmd/seeder/main.go init-rbac` to initialize system company and default role and default account, set the default account credentials in the .env
4. Login to the system company, create necessary subscription, plans etc
5. Create basic features
6. Create basic plan
7. Update company to include the new fields, assign the company the newly created plan
8. Then run `go run ./cmd/seeder/main.go assign-user-role` to assign a role to an existing account

## Related

- [[qubefini]]
- [[qubefini-multi-tenant-and-plans]]
- [[qubefini-deployment]]
