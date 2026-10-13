"027d702e-236f-cc28-e6d2-4fa4337abf48"	"CITIZEN"	"Ciudadano"		true

"03d4e37f-ed3f-4b94-f2cb-11caab2aaef3"	"COUNTER\_STAFF"	"Personal de ventanilla"		true

"70e3cfad-a422-ecf2-06f0-e7317f5d0461"	"COUNTER\_HEAD"	"Jefe de ventanilla"		true

"46893f06-71dc-91e3-eaab-4b23d7a7301b"	"DEPARTMENT\_HEAD"	"Jefe de departamento"		true



"AdministrativeStaff"	"AdministrativeStaff_pkey"	"PRIMARY KEY (""userId"")"
"AdministrativeStaff"	"AdministrativeStaff_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""User""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"Appointment"	"Appointment_branchId_procedureId_fkey"	"FOREIGN KEY (""branchId"", ""procedureId"") REFERENCES ""BranchProcedure""(""branchId"", ""procedureId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Appointment"	"Appointment_citizenId_fkey"	"FOREIGN KEY (""citizenId"") REFERENCES ""Citizen""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Appointment"	"Appointment_counterId_branchId_fkey"	"FOREIGN KEY (""counterId"", ""branchId"") REFERENCES ""ServiceCounter""(id, ""branchId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Appointment"	"Appointment_pkey"	"PRIMARY KEY (id)"
"Appointment"	"Appointment_slotId_branchId_fkey"	"FOREIGN KEY (""slotId"", ""branchId"") REFERENCES ""TimeSlot""(id, ""branchId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Attendance"	"Attendance_appointmentId_branchId_fkey"	"FOREIGN KEY (""appointmentId"", ""branchId"") REFERENCES ""Appointment""(id, ""branchId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Attendance"	"Attendance_counterId_branchId_fkey"	"FOREIGN KEY (""counterId"", ""branchId"") REFERENCES ""ServiceCounter""(id, ""branchId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Attendance"	"Attendance_pkey"	"PRIMARY KEY (id)"
"Attendance"	"Attendance_staffId_fkey"	"FOREIGN KEY (""staffId"") REFERENCES ""CounterStaff""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"AuthChallenge"	"AuthChallenge_pkey"	"PRIMARY KEY (id)"
"AuthChallenge"	"AuthChallenge_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""User""(id) ON UPDATE CASCADE ON DELETE CASCADE"
"Branch"	"Branch_counterHeadId_fkey"	"FOREIGN KEY (""counterHeadId"") REFERENCES ""CounterHead""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"Branch"	"Branch_municipalityId_fkey"	"FOREIGN KEY (""municipalityId"") REFERENCES ""Municipality""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"Branch"	"Branch_pkey"	"PRIMARY KEY (id)"
"BranchProcedure"	"BranchProcedure_branchId_fkey"	"FOREIGN KEY (""branchId"") REFERENCES ""Branch""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"BranchProcedure"	"BranchProcedure_pkey"	"PRIMARY KEY (""branchId"", ""procedureId"")"
"BranchProcedure"	"BranchProcedure_procedureId_fkey"	"FOREIGN KEY (""procedureId"") REFERENCES ""Procedure""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"Citizen"	"Citizen_municipalityId_fkey"	"FOREIGN KEY (""municipalityId"") REFERENCES ""Municipality""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"Citizen"	"Citizen_pkey"	"PRIMARY KEY (""userId"")"
"Citizen"	"Citizen_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""User""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"CounterHead"	"CounterHead_pkey"	"PRIMARY KEY (""userId"")"
"CounterHead"	"CounterHead_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""AdministrativeStaff""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"CounterStaff"	"CounterStaff_pkey"	"PRIMARY KEY (""userId"")"
"CounterStaff"	"CounterStaff_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""AdministrativeStaff""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"DepartmentHead"	"DepartmentHead_pkey"	"PRIMARY KEY (""userId"")"
"DepartmentHead"	"DepartmentHead_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""AdministrativeStaff""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"MailJob"	"MailJob_challengeId_fkey"	"FOREIGN KEY (""challengeId"") REFERENCES ""AuthChallenge""(id) ON UPDATE CASCADE ON DELETE CASCADE"
"MailJob"	"MailJob_pkey"	"PRIMARY KEY (id)"
"Municipality"	"Municipality_pkey"	"PRIMARY KEY (id)"
"Notification"	"Notification_appointmentId_fkey"	"FOREIGN KEY (""appointmentId"") REFERENCES ""Appointment""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"Notification"	"Notification_pkey"	"PRIMARY KEY (id)"
"Permission"	"Permission_pkey"	"PRIMARY KEY (id)"
"Procedure"	"Procedure_pkey"	"PRIMARY KEY (id)"
"ProcedureRecord"	"ProcedureRecord_attendanceId_fkey"	"FOREIGN KEY (""attendanceId"") REFERENCES ""Attendance""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"ProcedureRecord"	"ProcedureRecord_pkey"	"PRIMARY KEY (id)"
"Role"	"Role_pkey"	"PRIMARY KEY (id)"
"RolePermission"	"RolePermission_permissionId_fkey"	"FOREIGN KEY (""permissionId"") REFERENCES ""Permission""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"RolePermission"	"RolePermission_pkey"	"PRIMARY KEY (""roleId"", ""permissionId"")"
"RolePermission"	"RolePermission_roleId_fkey"	"FOREIGN KEY (""roleId"") REFERENCES ""Role""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"ServiceCounter"	"ServiceCounter_branchId_fkey"	"FOREIGN KEY (""branchId"") REFERENCES ""Branch""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"ServiceCounter"	"ServiceCounter_pkey"	"PRIMARY KEY (id)"
"Session"	"Session_pkey"	"PRIMARY KEY (id)"
"Session"	"Session_userId_fkey"	"FOREIGN KEY (""userId"") REFERENCES ""User""(id) ON UPDATE CASCADE ON DELETE CASCADE"
"StaffAssignment"	"StaffAssignment_counterId_fkey"	"FOREIGN KEY (""counterId"") REFERENCES ""ServiceCounter""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"StaffAssignment"	"StaffAssignment_pkey"	"PRIMARY KEY (id)"
"StaffAssignment"	"StaffAssignment_staffId_fkey"	"FOREIGN KEY (""staffId"") REFERENCES ""CounterStaff""(""userId"") ON UPDATE CASCADE ON DELETE RESTRICT"
"TimeSlot"	"TimeSlot_branchId_fkey"	"FOREIGN KEY (""branchId"") REFERENCES ""Branch""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"TimeSlot"	"TimeSlot_pkey"	"PRIMARY KEY (id)"
"User"	"User_pkey"	"PRIMARY KEY (id)"
"User"	"User_roleId_fkey"	"FOREIGN KEY (""roleId"") REFERENCES ""Role""(id) ON UPDATE CASCADE ON DELETE RESTRICT"
"_prisma_migrations"	"_prisma_migrations_pkey"	"PRIMARY KEY (id)"



"AdministrativeStaff"	"userId"	"uuid"	"NO"	

"AdministrativeStaff"	"street"	"character varying"	"NO"	

"AdministrativeStaff"	"exteriorNumber"	"character varying"	"NO"	

"AdministrativeStaff"	"interiorNumber"	"character varying"	"YES"	

"AdministrativeStaff"	"neighborhood"	"character varying"	"NO"	

"AdministrativeStaff"	"postalCode"	"character varying"	"NO"	

"AdministrativeStaff"	"socialSecurityNumber"	"character varying"	"NO"	

"Branch"	"id"	"uuid"	"NO"	

"Branch"	"municipalityId"	"uuid"	"NO"	

"Branch"	"counterHeadId"	"uuid"	"YES"	

"Branch"	"name"	"character varying"	"NO"	

"Branch"	"type"	"character varying"	"NO"	

"Branch"	"street"	"character varying"	"YES"	

"Branch"	"exteriorNumber"	"character varying"	"YES"	

"Branch"	"interiorNumber"	"character varying"	"YES"	

"Branch"	"neighborhood"	"character varying"	"YES"	

"Branch"	"postalCode"	"character varying"	"YES"	

"Branch"	"latitude"	"numeric"	"YES"	

"Branch"	"longitude"	"numeric"	"YES"	

"Branch"	"active"	"boolean"	"NO"	"true"

"Branch"	"createdAt"	"timestamp without time zone"	"NO"	"CURRENT\_TIMESTAMP"

"Branch"	"updatedAt"	"timestamp without time zone"	"NO"	

"Branch"	"address"	"text"	"YES"	

"Branch"	"phone"	"character varying"	"YES"	

"Branch"	"email"	"character varying"	"YES"	

"Branch"	"officeHours"	"character varying"	"YES"	

"Branch"	"sourceKey"	"character varying"	"YES"	

"Branch"	"sourceUrl"	"text"	"YES"	

"Branch"	"sourceSha256"	"character"	"YES"	

"Branch"	"sourceCheckedAt"	"timestamp without time zone"	"YES"	

"Branch"	"published"	"boolean"	"NO"	"false"

"CounterHead"	"userId"	"uuid"	"NO"	

"DepartmentHead"	"userId"	"uuid"	"NO"	

"Municipality"	"id"	"uuid"	"NO"	

"Municipality"	"code"	"character varying"	"NO"	

"Municipality"	"name"	"character varying"	"NO"	

"Permission"	"id"	"uuid"	"NO"	

"Permission"	"code"	"character varying"	"NO"	

"Permission"	"description"	"text"	"NO"	

"Role"	"id"	"uuid"	"NO"	

"Role"	"code"	"character varying"	"NO"	

"Role"	"name"	"character varying"	"NO"	

"Role"	"hierarchyLevel"	"integer"	"YES"	

"Role"	"active"	"boolean"	"NO"	"true"

"RolePermission"	"roleId"	"uuid"	"NO"	

"RolePermission"	"permissionId"	"uuid"	"NO"	

"ServiceCounter"	"id"	"uuid"	"NO"	

"ServiceCounter"	"branchId"	"uuid"	"NO"	

"ServiceCounter"	"number"	"integer"	"NO"	

"ServiceCounter"	"active"	"boolean"	"NO"	"true"

"StaffAssignment"	"id"	"uuid"	"NO"	

"StaffAssignment"	"staffId"	"uuid"	"NO"	

"StaffAssignment"	"counterId"	"uuid"	"NO"	

"StaffAssignment"	"startsAt"	"timestamp without time zone"	"NO"	

"StaffAssignment"	"endsAt"	"timestamp without time zone"	"YES"	

"User"	"id"	"uuid"	"NO"	

"User"	"firstName"	"character varying"	"NO"	

"User"	"lastName"	"character varying"	"NO"	

"User"	"curp"	"character varying"	"NO"	

"User"	"email"	"character varying"	"NO"	

"User"	"passwordHash"	"text"	"NO"	

"User"	"municipality"	"character varying"	"NO"	

"User"	"phone"	"character varying"	"YES"	

"User"	"rfc"	"character varying"	"YES"	

"User"	"emailVerifiedAt"	"timestamp without time zone"	"YES"	

"User"	"active"	"boolean"	"NO"	"true"

"User"	"createdAt"	"timestamp without time zone"	"NO"	"CURRENT\_TIMESTAMP"

"User"	"updatedAt"	"timestamp without time zone"	"NO"	

"User"	"municipalityCode"	"character varying"	"YES"	

"User"	"roleId"	"uuid"	"NO"	

"User"	"phoneCountryCode"	"character varying"	"YES"	

"User"	"paternalLastName"	"character varying"	"YES"	

"User"	"maternalLastName"	"character varying"	"YES"	

"Appointment"	"id"	"uuid"	"NO"	
"Appointment"	"folio"	"character varying"	"NO"	
"Appointment"	"citizenId"	"uuid"	"NO"	
"Appointment"	"branchId"	"uuid"	"NO"	
"Appointment"	"procedureId"	"uuid"	"NO"	
"Appointment"	"slotId"	"uuid"	"NO"	
"Appointment"	"counterId"	"uuid"	"YES"	
"Appointment"	"requestedAt"	"timestamp without time zone"	"NO"	"CURRENT_TIMESTAMP"
"Appointment"	"scheduledAt"	"timestamp without time zone"	"NO"	
"Appointment"	"status"	"character varying"	"NO"	
"Appointment"	"qrToken"	"text"	"NO"	
"Appointment"	"occupiesCapacity"	"boolean"	"NO"	"true"
"Appointment"	"updatedAt"	"timestamp without time zone"	"NO"	
"Attendance"	"id"	"uuid"	"NO"	
"Attendance"	"appointmentId"	"uuid"	"NO"	
"Attendance"	"branchId"	"uuid"	"NO"	
"Attendance"	"staffId"	"uuid"	"NO"	
"Attendance"	"counterId"	"uuid"	"NO"	
"Attendance"	"arrivedAt"	"timestamp without time zone"	"NO"	
"Attendance"	"qrValidatedAt"	"timestamp without time zone"	"YES"	
"Attendance"	"startedAt"	"timestamp without time zone"	"YES"	
"Attendance"	"endedAt"	"timestamp without time zone"	"YES"	
"Attendance"	"late"	"boolean"	"NO"	
"BranchProcedure"	"branchId"	"uuid"	"NO"	
"BranchProcedure"	"procedureId"	"uuid"	"NO"	
"BranchProcedure"	"active"	"boolean"	"NO"	"true"
"Procedure"	"id"	"uuid"	"NO"	
"Procedure"	"name"	"character varying"	"NO"	
"Procedure"	"category"	"character varying"	"YES"	
"Procedure"	"requirements"	"text"	"YES"	
"Procedure"	"cost"	"numeric"	"YES"	
"Procedure"	"estimatedMinutes"	"integer"	"YES"	
"Procedure"	"active"	"boolean"	"NO"	"true"
"Procedure"	"createdAt"	"timestamp without time zone"	"NO"	"CURRENT_TIMESTAMP"
"Procedure"	"updatedAt"	"timestamp without time zone"	"NO"	
"Procedure"	"description"	"text"	"YES"	
"Procedure"	"sourceKey"	"character varying"	"YES"	
"Procedure"	"sourceUrl"	"text"	"YES"	
"Procedure"	"sourceSha256"	"character"	"YES"	
"Procedure"	"sourceCheckedAt"	"timestamp without time zone"	"YES"	
"Procedure"	"published"	"boolean"	"NO"	"false"
"ProcedureRecord"	"id"	"uuid"	"NO"	
"ProcedureRecord"	"attendanceId"	"uuid"	"NO"	
"ProcedureRecord"	"registeredAt"	"timestamp without time zone"	"NO"	"CURRENT_TIMESTAMP"
"ProcedureRecord"	"finalStatus"	"character varying"	"NO"	
"ProcedureRecord"	"observations"	"text"	"YES"	
"TimeSlot"	"id"	"uuid"	"NO"	
"TimeSlot"	"branchId"	"uuid"	"NO"	
"TimeSlot"	"date"	"date"	"NO"	
"TimeSlot"	"startsAt"	"time without time zone"	"NO"	
"TimeSlot"	"endsAt"	"time without time zone"	"NO"	
"TimeSlot"	"capacity"	"integer"	"NO"	
"TimeSlot"	"occupied"	"integer"	"NO"	"0"
"TimeSlot"	"active"	"boolean"	"NO"	"true"