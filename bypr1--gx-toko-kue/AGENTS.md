# GitHub Copilot Custom Instructions for Go Backend Service

## Role Instructions
You are a Go backend service developer in globalxtreme. You are responsible for implementing boilerplate and helping developers follow the coding standards and architecture patterns established in the project. You will provide code suggestions, explanations, and best practices for building scalable and maintainable backend services. Everytime you suggest code, you must follow the existing architecture patterns, use the established error handling mechanisms, leverage the custom framework utilities, maintain consistency with existing code style, consider performance and scalability, include proper validation and error handling, and use appropriate logging where necessary. Ensure always use core library for common operations and utilities which can be found in `github.com/globalxtreme/go-core/v2`.

***"Important"*** Don't leave any comments in the code you suggest, unless it is a comment that is already in the codebase. Do not add any comments that are not already in the codebase.

## Project Overview
This is a Go backend service built with a layered architecture following Domain-Driven Design principles. The service uses gRPC, REST APIs, message queues (RabbitMQ), and various third-party integrations.

## Architecture & Patterns

### Project Structure
- **cmd/**: Entry points and CLI commands using Cobra
- **internal/**: Private application code
  - **app/**: Application layer (API routes, gRPC servers, console commands, queues). Entry points for the application.
    - **api/**: HTTP API handlers
      - **router.go**: Main router for API endpoints
      - ***/** : Specific API types (web, mobile, private)
        - **router.go**: Router for specific API type
        - **handler/**: Handlers for API endpoints
    - **console/**: CLI commands
      - **command/**: Command implementations
      - **kernel.go**: Command registration
    - **database/**: Database migrations and seeding
      - **migration/**: Database migration files
      - **seeder/**: Database seeder files
      - **migrate.go**: Migration registration
      - **seeder.go**: Seeder registration
    - **grpc/**: gRPC server implementations
      - **server/**: gRPC logic handlers
      - **worker.go**: gRPC worker registration
    - **privateapi/**: Private API handlers
      - **handler/**: Handlers for private API endpoints
      - **router.go**: Router for private API endpoints
    - **queue/**: Internal message queue handlers using redis
      - **job/**: Queue job implementations
      - **worker.go**: Queue worker registration
    - **rabbitmq/**: RabbitMQ message queue handlers
  - **pkg/**: Domain/business logic packages
    - **config/**: Configuration management
    - **constant/**: Application constants
    - **core/**: Core utilities and helpers
    - **encrypt/**: Encryption and decryption utilities
    - **error/**: Custom error types and handling
    - **form/**: Interfaces for form transmission
    - **grpc/**: gRPC generated code and utilities
    - **layout/**: Layout templates for HTML rendering for emails and pdfs
    - **middleware/**: HTTP middleware for accessing services
    - **model/**: Database models using GORM
    - **number/**: Number utilities
    - **parser/**: Data transformation utilities usually for API responses
    - **port/**: Repository interfaces and ports
    - **saga/**: Saga pattern implementations for distributed transactions
      - **grpc/**: gRPC saga implementations
      - **privateapi/**: Private API saga implementations
    - **thirdparty/**: Integrations with third-party api services such as Email, Telegram, etc.
    - ***/**: Domain-specific packages, hooks and helpers
  - ***/**: Domain-specific code that only applies to the specific domain
    - **repository/**: Data access layer for the domain
    - **service/**: Business logic for the domain
    - **mail/**: Email sending and templating
    - **excel/**: Excel file processing and generation
    - **pdf/**: PDF generation
- **storage/**: File storage and logs
  - **app/**: Application-specific storage
  - **logs/**: Application logs
- **stubs/**: Code generation templates

### Layered Architecture
Follow this pattern when creating new features:
1. **Model** (`internal/pkg/model/`): Database entities using GORM
2. **Repository** (`internal/*/repository/`): Data access layer
3. **Service** (`internal/*/service/`): Business logic
4. **Handler** (`internal/app/*/handler/`): HTTP/gRPC request handlers
5. **Parser** (`internal/pkg/parser/`): Data transformation
6. **Form** (`internal/pkg/form/`): Request validation

## Coding Standards

### Go Conventions
- Use PascalCase for exported functions, types, and methods
- Use camelCase for unexported functions and variables
- Follow standard Go project layout
- Use meaningful package names that reflect their purpose

### Database Models
- Extend one of these models for all entities
	- `xtrememodel.BaseModel` for common use case
	- `xtrememodel.BaseModelUUID` for UUID primary keys
	- `xtrememodel.BaseModelWithoutID` for model that don't use default ID (like composite keys, or else)
- Use GORM tags for database mapping
- Filename should be in PascalCase and match the model struct name, e.g., `User.go`
- Implement `TableName()` method for custom table names
- Implement `SetReference()` method for reference handling (used in activity logging)
- Struct attributes should use GORM tags for column definitions
- Struct attributes that define foreign Id should be named as `<ModelName>Id`, e.g., `UserId` for `User` model
- Order of the attributes in the struct should be as follows:
  - Between `xtrememodel.BaseModel`, `xtrememodel.BaseModelUUID`, or `xtrememodel.BaseModelWithoutID`
  - Foreign Ids
  - Required attributes
  - Optional attributes
  - Relationships (e.g., `Posts []UserPosts`)
- If there is custom data type for a field, it should be implemented with following:
	- For `[]map[string]interface{}}` use `xtrememodel.ArrayMapInterfaceColumn`
	- For `map[string]interface{}` use `xtrememodel.MapInterfaceColumn`
- Table columns should be named in camelCase, e.g., `userId`, `otherAttributes`, `createdAt`
- Table names should be pluralized and use snake_case, e.g., `users`, `posts`, `comments`

Example model files:
```go
package model

import (
	xtrememodel "github.com/globalxtreme/go-core/v2/model"
)

type User struct {
	xtrememodel.BaseModel
	Name       string  `gorm:"column:name;not null"`
	Avatar     *string  `gorm:"column:avatar;"`
	Age       int     `gorm:"column:age;type:int;default:0"`
	OtherAttributes xtrememodel.MapInterfaceColumn `gorm:"column:otherAttributes;type:json"`

	Posts    []Post    `gorm:"foreignKey:UserId"`
	Comments []Comment  `gorm:"foreignKey:UserId"`
}

func (User) TableName() string {
	return "users"
}

func (md User) SetReference() uint {
	return md.BaseModel.ID
}

```
```go
package model

import (
	xtrememodel "github.com/globalxtreme/go-core/v2/model"
)

type Post struct {
	xtrememodel.BaseModel
	UserId   uint    `gorm:"column:userId;not null"`
	Title     string  `gorm:"column:title;not null"`
	Content   string  `gorm:"column:content;not null"`
	Properties xtrememodel.ArrayMapInterfaceColumn `gorm:"column:properties;type:json"`

	Comments []Comment `gorm:"foreignKey:PostId"`
}

func (Post) TableName() string {
	return "posts"
}

func (md Post) SetReference() uint {
	return md.BaseModel.ID
}
```
```go
package model

import (
	xtrememodel "github.com/globalxtreme/go-core/v2/model"
)

type Comment struct {
	xtrememodel.BaseModel
	UserId  uint    `gorm:"column:userId;not null"`
	PostId  uint    `gorm:"column:postId;not null"`
	Content string  `gorm:"column:content;not null"`
}

func (Comment) TableName() string {
	return "comments"
}

func (md Comment) SetReference() uint {
	return md.BaseModel.ID
}
```

### Database Migrations
- When creating new migrations, *YOU MUST ALWAYS* use cli command: `go run ./cmd/main.go gen:migration <NAME>`
- For migration that creates new tables, use `<FEATURE_NAME>` as the `<NAME>`, e.g., `Product`
- For migration that modifies existing tables, follow pattern of files inside `internal/app/database/migration/`, if none just use what you use for creating new tables
- Then register migrations in `internal/app/database/migrate.go`
- After new migration is created, you should implement the file content following the example below.

Example migration for new table:
```go
package migration

import (
	"os"
	"service/internal/pkg/config"
	"service/internal/pkg/model"

	xtremedb "github.com/globalxtreme/go-core/v2/database"
)

type User_1726651240922259 struct{}

func (User_1726651240922259) Reference() string {
	return "User_1726651240922259"
}

func (User_1726651240922259) Tables() []xtremedb.Table {
	owner := os.Getenv("DB_OWNER")

	return []xtremedb.Table{
		{Connection: config.PgSQL, CreateTable: model.User{}, Owner: owner},
		{Connection: config.PgSQL, CreateTable: model.UserProfile{}, Owner: owner},
		{Connection: config.PgSQL, CreateTable: model.UserSettings{}, Owner: owner},
	}
}

func (User_1726651240922259) Columns() []xtremedb.Column {
	return []xtremedb.Column{}
}

```

Example migration for modifying existing table:
```go
package migration

import (
	"os"
	"service/internal/pkg/config"
	"service/internal/pkg/model"

	xtremedb "github.com/globalxtreme/go-core/v2/database"
)

type Post_1738380774908382 struct{}

func (Post_1738380774908382) Reference() string {
	return "Post_1738380774908382"
}

func (Post_1738380774908382) Tables() []xtremedb.Table {
	owner := os.Getenv("DB_OWNER")
	
	return []xtremedb.Table{
		{Connection: config.PgSQL, RenameTable: xtremedb.Rename{Old: "posts", New: "threads"}, Owner: owner},
	}
}

func (Post_1738380774908382) Columns() []xtremedb.Column {
	return []xtremedb.Column{
		{
			Connection:  config.PgSQL,
			Model:       model.Comment{},
			DropColumns: []string{"column1", "column2"},
			RenameColumns: []xtremedb.Rename{
				{
					Old: "oldColumnName",
					New: "newColumnName",
				},
			},
			AddColumns: []string{"newColumn1", "newColumn2"},
			AlterColumns: []string{"column3", "column4"},
		},
	}
}
```

### Database Seeder
- Put in `internal/app/database/seeder/`
- Implement seeders for initial data population
- Register seeders in `internal/app/database/seeder.go`

Example seeder:
```go
package seeder

import (
	"service/internal/pkg/model"

	"gorm.io/gorm"
)

type UserSeeder struct {}

func (seed *UserSeeder) Seed() {
	users := seed.setUsersData()
	for _, user := range users {
		var count int64
		config.PgSQL.Model(&model.User{}).Where("email = ?", user["email"]).Count(&count)
		if count > 0 {
			continue
		}

		config.PgSQL.Model(&model.User{}).Create(&model.User{
			Email:    user["email"].(string),
			Password: user["password"].(string),
			FullName: user["full_name"].(string),
			Age:     user["age"].(int),
		})
	}
}

// --- UNEXPORTED FUNCTIONS ---

func (seed *UserSeeder) setUsersData() []map[string]interface{} {
	return []map[string]interface{}{
		{
			"email":    "john.doe@example.com",
			"password": "password123",
			"full_name": "John Doe",
			"age":     30,
		},
		{
			"email":    "jane.smith@example.com",
			"password": "password456",
			"full_name": "Jane Smith",
			"age":     25,
		},
	}
}
```

### Form Request
- Use `internal/pkg/form/` for request validation
- Implement form validation using `github.com/go-playground/validator/v10`
- Json attributes should be in camelCase, e.g., `name` instead of `Name`, `sellPrice` instead of `sell_price`

Example form structure (for json request):
```go
package form

import (
	"net/http"
	"service/internal/pkg/core"

	xtrememdw "github.com/globalxtreme/go-core/v2/middleware"
)

type UserForm struct {
	Email    string `json:"email" validate:"required,email"`
	Password string `json:"password" validate:"required,min=8"`
	FullName string `json:"fullName" validate:"required,max=100"`
	Age     int    `json:"age" validate:"required,gte=0"`
	Contacts []UserContactForm `json:"contacts" validate:"required,dive"`
}

type UserContactForm struct {
	ID *uint   `json:"id"`
	Type string `json:"type" validate:"required"`
	Value string `json:"value" validate:"required"`
	Primary bool   `json:"primary" validate:"required"`
	Deleted bool   `json:"deleted"`
}

func (f *UserForm) Validate() {
	va := xtrememdw.Validator{}
	va.Make(f)
}

func (f *UserForm) APIParse(r *http.Request) {
	form := core.BaseForm{}
	form.APIParse(r, &f)
}

```

Example form structure (for form request):
```go
package form

import (
	"net/http"
	"service/internal/pkg/core"

	xtrememdw "github.com/globalxtreme/go-core/v2/middleware"
)

type UserForm struct {
	Request *http.Request `form:"-"`
	Email    string `form:"email" validate:"required,email"`
	Password string `form:"password" validate:"required,min=8"`
	FullName string `form:"fullName" validate:"required,max=100"`
	Age     int    `form:"age" validate:"required,gte=0"`
	Contacts []UserContactForm `form:"contacts" validate:"required,dive"`
}

type UserContactForm struct {
	ID *uint   `form:"id"`
	Type string `form:"type" validate:"required"`
	Value string `form:"value" validate:"required"`
	Primary bool   `form:"primary" validate:"required"`
	Deleted bool   `form:"deleted"`
}

func (f *UserForm) Validate() {
	va := xtrememdw.Validator{}
	va.Make(f)
}

func (f *UserForm) APIParse(r *http.Request) {
	f.Request = r
	form := core.BaseForm{}
	form.APIMultipartFormParse(r, &f)
}

```

### Response Parser
- Use `internal/pkg/parser/` for data transformation
- Implement parsers for API responses
- Returned attributes should be in camelCase `sellPrice` instead of `sell_price`
- Functions that should be implemented in order:
  - `Briefs()`: Returns a brief list of objects
  - `Brief()`: Returns a brief object
  - `Get()`: Returns a detailed list of objects
  - `First()`: Returns a detailed object
  - `CreateActivity(action string)`: Returns the object for create activity
  - `DeleteActivity(action string)`: Returns the object for delete activity
  - `GeneralActivity(action string)`: Returns the object for general activity
  - `UpdateActivity(action string)`: Returns the object for update activity

Example parser structure:
```go
package parser

import (
	"service/internal/pkg/model"
)

type UserParser struct {
	Array  []model.User
	Object model.User
}

func (parser UserParser) Briefs() []interface{} {
	var result []interface{}
	for _, obj := range parser.Array {
		result = append(result, UserParser{Object: obj}.Brief())
	}
	return result
}

func (parser UserParser) Brief() interface{} {
	user := parser.Object
	return map[string]interface{}{
		"id":       user.ID,
		"email":    user.Email,
		"fullName": user.FullName,
		"age":      user.Age,
	}
}

func (parser UserParser) Get() []interface{} {
	var result []interface{}
	for _, obj := range parser.Array {
		result = append(result, UserParser{Object: obj}.First())
	}
	return result
}

func (parser UserParser) First() interface{} {
	user := parser.Object
	var contacts []map[string]interface{}
	for _, contact := range user.Contacts {
		contacts = append(contacts, map[string]interface{}{
			"type":    contact.Type,
			"value":   contact.Value,
			"primary": contact.Primary,
		})
	}
	return map[string]interface{}{
		"id":       user.ID,
		"email":    user.Email,
		"fullName": user.FullName,
		"age":      user.Age,
		"contacts": contacts,
	}
}

func (parser UserParser) CreateActivity(action string) interface{} {
	return parser.First()
}

func (parser UserParser) DeleteActivity(action string) interface{} {
	return parser.First()
}

func (parser UserParser) GeneralActivity(action string) interface{} {
	return parser.First()
}

func (parser UserParser) UpdateActivity(action string) interface{} {
	return parser.First()
}
```

### Domain Specific Codes
- Place domain-specific code in `internal/<domain>/`
- Domain cannot interact directly with other domains, use either `internal/pkg/` or expose demanded repository/service function interfaces inside `internal/pkg/port`

- Use `repository/` for data access layer

Example repository structure:
```go
package repository

import (
	"net/url"
	"service/internal/pkg/config"
	"service/internal/pkg/core"
	erro "service/internal/pkg/error"
	"service/internal/pkg/form"
	"service/internal/pkg/model"

	xtrememodel "github.com/globalxtreme/go-core/v2/model"
	"gorm.io/gorm"
)

type UserRepository interface {
	core.UserRepository
	core.PaginateRepository[model.User]
	core.FirstIdRepository[model.User]

	APIPreload(query *gorm.DB) *gorm.DB

	Store(form form.UserForm) model.User
	Delete(user model.User)
	Update(user model.User, form form.UserForm) model.User
	UpdateAvatar(user model.User, avatar string) model.User
	SaveContacts(user model.User, contacts []form.UserContactForm) []model.UserContact
	DeleteContacts(user model.User)
}

func NewUserRepository(args ...*gorm.DB) UserRepository {
	repository := userRepository{}
	if len(args) > 0 {
		repository.transaction = args[0]
	}

	return &repository
}

type userRepository struct {
	transaction *gorm.DB
}

func (repo *userRepository) SetTransaction(tx *gorm.DB) {
	repo.transaction = tx
}

func (repo *userRepository) APIPreload(query *gorm.DB) *gorm.DB {
	return query.Preload("Contacts")
}

func (repo *userRepository) FirstById(id any, args ...func(query *gorm.DB) *gorm.DB) model.User {
	var user model.User
	query := config.PgSQL

	if len(args) > 0 {
		query = args[0](query)
	}

	err := query.First(&user, "id = ?", id).Error
	if err != nil {
		erro.ErrXtremeUserGet(err.Error())
	}

	return user
}

func (repo *userRepository) Paginate(parameter url.Values) ([]model.User, interface{}, error) {
	fromDate, toDate := core.SetDateRange(parameter)

	query := config.PgSQL.Where("\"createdAt\" BETWEEN ? AND ?", fromDate, toDate)

	if search := parameter.Get("search"); len(search) > 3 {
		query = query.Where("email ILIKE ? OR \"fullName\" ILIKE ?", "%"+search+"%", "%"+search+"%")
	}

	users, pagination, err := xtrememodel.Paginate(query.Order("id DESC"), parameter, model.User{})
	if err != nil {
		return nil, nil, err
	}

	return users, pagination, nil
}

func (repo *userRepository) Store(form form.UserForm) model.User {
	user := model.User{
		Email:    form.Email,
		Password: form.Password,
		FullName: form.FullName,
		Age:      form.Age,
	}

	err := repo.transaction.Create(&user).Error
	if err != nil {
		erro.ErrXtremeUserSave(err.Error())
	}

	return user
}

func (repo *userRepository) Update(user model.User, form form.UserForm) model.User {
	user.Email = form.Email
	user.FullName = form.FullName
	user.Age = form.Age

	err := repo.transaction.Save(&user).Error
	if err != nil {
		erro.ErrXtremeUserSave(err.Error())
	}

	return user
}

func (repo *userRepository) UpdateAvatar(user model.User, avatar string) model.User {
	user.Avatar = &avatar
	err := repo.transaction.Save(&user).Error
	if err != nil {
		erro.ErrXtremeUserSave(err.Error())
	}
	return user
}

func (repo *userRepository) Delete(user model.User) {
	err := repo.transaction.Delete(&user).Error
	if err != nil {
		erro.ErrXtremeUserDelete(err.Error())
	}
}

func (repo *userRepository) SaveContacts(user model.User, contacts []form.UserContactForm) []model.UserContact {
	var userContacts []model.UserContact

	for _, request := range contacts {
		var contact model.UserContact
		if request.ID != nil && *request.ID > 0 {
			if request.Deleted {
				err := repo.transaction.Where("id = ?", *request.ID).Delete(&model.UserContact{}).Error
				if err != nil {
					erro.ErrXtremeUserContactDelete(err.Error())
				}
			} else {
				repo.transaction.First(&contact, "id = ?", *request.ID)
				if contact.ID == 0 {
					erro.ErrXtremeUserContactGet("ID not found")
				}

				contact.Type = request.Type
				contact.Value = request.Value
				contact.Primary = request.Primary
				err := repo.transaction.Save(&contact).Error
				if err != nil {
					erro.ErrXtremeUserContactSave(err.Error())
				}
			}
		} else {
			contact = model.UserContact{
				UserId:  user.ID,
				Type:    request.Type,
				Value:   request.Value,
				Primary: request.Primary,
			}
			err := repo.transaction.Save(&contact).Error
			if err != nil {
				erro.ErrXtremeUserContactSave(err.Error())
			}
		}

		if contact.ID > 0 {
			userContacts = append(userContacts, contact)
		}
	}

	return userContacts
}

func (repo *userRepository) DeleteContacts(user model.User) {
	err := repo.transaction.Where("\"userId\" = ?", user.ID).Delete(&model.UserContact{}).Error
	if err != nil {
		erro.ErrXtremeUserContactDelete(err.Error())
	}
}
```

- Use `service/` for business logic
  - Every service that would modify the data (Create/Update/Delete) should use a transaction and save their action as `Activity`


Example service structure for CRUD:
```go
package service

import (
	"fmt"
	"service/internal/pkg/activity"
	"service/internal/pkg/config"
	"service/internal/pkg/constant"
	"service/internal/pkg/form"
	"service/internal/pkg/model"
	"service/internal/pkg/parser"
	"service/internal/pkg/port"
	"service/internal/user/excel"
	"service/internal/user/repository"

	"gorm.io/gorm"
)

type UserService interface {
	SetMembershipRepository(repo port.MembershipRepository)

	Create(form form.UserForm) model.User
	Update(form form.UserForm, id string) model.User
	Delete(id string) bool
	ReportExcel(form form.UserReportForm) string
}

func NewUserService() UserService {
	return &userService{}
}

type userService struct {
	repository           repository.UserRepository
	membershipRepository port.MembershipRepository
}

func (srv *userService) SetMembershipRepository(repo port.MembershipRepository) {
	srv.membershipRepository = repo
}

func (srv *userService) Create(form form.UserForm) model.User {
	var user model.User
	srv.prepare()

	config.PgSQL.Transaction(func(tx *gorm.DB) error {
		srv.setRepositoriesWithTransaction(tx)

		memberships := srv.getMemberships(form.Memberships)
		totalAmount := srv.calculateTotalPrice(form.Memberships, memberships)

		user = srv.repository.Store(form, totalAmount)
		user.Memberships = srv.repository.SaveMemberships(user, form.Memberships, memberships)

		activity.UseActivity{}.SetReference(user).SetParser(&parser.UserParser{Object: user}).SetNewProperty(constant.ACTION_CREATE).
			Save(fmt.Sprintf("Created new user [%d] with total amount %.2f", user.ID, user.TotalPrice))

		return nil
	})

	return user
}

func (srv *userService) Update(form form.UserForm, id string) model.User {
	user := srv.prepareWithData(id)

	config.PgSQL.Transaction(func(tx *gorm.DB) error {
		srv.setRepositoriesWithTransaction(tx)

		act := activity.UseActivity{}.
			SetReference(user).
			SetParser(&parser.UserParser{Object: user}).
			SetOldProperty(constant.ACTION_UPDATE)

		memberships := srv.getMemberships(form.Memberships)
		totalAmount := srv.calculateTotalPrice(form.Memberships, memberships)

		user = srv.repository.Update(user, form, totalAmount)
		user.Memberships = srv.repository.SaveMemberships(user, form.Memberships, memberships)

		act.SetParser(&parser.UserParser{Object: user}).
			SetNewProperty(constant.ACTION_UPDATE).
			Save(fmt.Sprintf("Updated user [%d] with total amount %.2f", user.ID, user.TotalPrice))

		return nil
	})

	return user
}

func (srv *userService) Delete(id string) bool {
	user := srv.prepareWithData(id)

	config.PgSQL.Transaction(func(tx *gorm.DB) error {
		srv.setRepositoriesWithTransaction(tx)

		srv.repository.DeleteMemberships(user)
		srv.repository.Delete(user)

		activity.UseActivity{}.SetReference(user).SetParser(&parser.UserParser{Object: user}).SetOldProperty(constant.ACTION_DELETE).
			Save(fmt.Sprintf("Deleted user [%d]", user.ID))

		return nil
	})
	return true
}

func (srv *userService) ReportExcel(form form.UserReportForm) string {
	srv.prepare()

	users := srv.repository.FindForReport(form)
	userExcel := excel.UserExcel{
		Users: users,
	}

	filename, _ := userExcel.Generate()
	return filename
}

func (srv *userService) prepare() {
	srv.repository = repository.NewUserRepository(nil)
}

func (srv *userService) prepareWithData(id string) model.User {
	srv.prepare()
	return srv.repository.FirstById(id)
}

func (srv *userService) setRepositoriesWithTransaction(tx *gorm.DB) {
	srv.repository.SetTransaction(tx)
	srv.membershipRepository.SetTransaction(tx)
}

func (srv *userService) getMemberships(items []form.UserMembershipForm) map[uint]model.Membership {
	var membershipIDs []any
	for _, it := range items {
		membershipIDs = append(membershipIDs, it.MembershipId)
	}

	memberships := srv.membershipRepository.FindByIds(membershipIDs)
	membershipMap := make(map[uint]model.Membership)
	for _, membership := range memberships {
		membershipMap[membership.ID] = membership
	}
	return membershipMap	
}

func (srv *userService) calculateTotalPrice(items []form.UserMembershipForm, membershipMap map[uint]model.Membership) float64 {
	var totalAmount float64
	for _, it := range items {
		if membership, exists := membershipMap[it.MembershipId]; exists {
			totalAmount += float64(it.Quantity) * membership.Price
		}
	}
	return totalAmount
}

```

- (Optional, only append if requested) Use `mail/` for email sending and templating
- (Optional, only append if requested) Use `excel/` for Excel file processing and generation
example excel structure:
```go
package excel

import (
	xtremecore "github.com/globalxtreme/go-core/v2"
	xtremepkg "github.com/globalxtreme/go-core/v2/pkg"
	"github.com/xuri/excelize/v2"
	"service/internal/pkg/constant"
)

type TestingExcel struct {
}

func (ex TestingExcel) Generate() error {
	sheets, properties := ex.setSheetsAndProperties()

	excel := ex.newFile(sheets, properties)
	excel = ex.modifySheet(excel)

	err := excel.Save(constant.PathExcelTesting(), "testing.xlsx")
	if err != nil {
		return err
	}

	return nil
}

func (ex TestingExcel) newFile(sheets []string, properties [][][]interface{}) xtremecore.Excel {
	excel := xtremecore.Excel{
		Sheets:     sheets,
		Properties: properties,
		IsPublic:   false,
	}

	excel.NewFile()

	return excel
}

func (ex TestingExcel) modifySheet(excel xtremecore.Excel) xtremecore.Excel {
	excel.SetStyle(&excelize.Style{
		Border: []excelize.Border{
			{Type: "left", Color: "860A35", Style: 1},
			{Type: "right", Color: "860A35", Style: 1},
			{Type: "top", Color: "860A35", Style: 1},
			{Type: "bottom", Color: "860A35", Style: 1},
		},
	}, "A1:C1")

	excel.SetStyle(&excelize.Style{
		Border: []excelize.Border{
			{Type: "left", Color: "A3B763", Style: 2},
			{Type: "right", Color: "A3B763", Style: 2},
			{Type: "top", Color: "A3B763", Style: 2},
			{Type: "bottom", Color: "A3B763", Style: 2},
		},
	}, "A3:C4")

	excel.MergeCells("A5:C5", "A6:A7")

	excel.SetWidthCols([]xtremecore.ColWidth{
		{Cells: "A", Width: 5},
		{Cells: "B", Width: 15},
		{Cells: "C", Width: 5},
	})

	excel.SetHeightRows([]xtremecore.RowHeight{
		{Row: 1, Height: 15},
	})

	return excel
}

func (TestingExcel) setSheetsAndProperties() ([]string, [][][]interface{}) {
	var sheets []string
	var properties [][][]interface{}

	sheets = append(sheets, "Testing 1", "Testing 2")
	for _ = range sheets {
		properties = append(properties, [][]interface{}{
			{"ID", "Name", "Age"},
		})
	}

	dataProperties := [][]int{
		{29, 23, 12},
		{19, 26, 14},
	}

	for sKey, property := range dataProperties {
		for pKey, val := range property {
			properties[sKey] = append(properties[sKey], []interface{}{pKey + 1, xtremepkg.RandomString(10), val})
		}
	}

	return sheets, properties
}
```

- (Optional, only append if requested) Use `pdf/` for PDF generation

### API Route Handlers
- When creating new handlers, *YOU MUST ALWAYS* use cli command: `go run ./cmd/main.go gen:handler <NAME> --type=<web/mobile> --resource`
- Use `--type=web` for web API handlers, `--type=mobile` for mobile API handlers
- Use `--resource` flag to generate resourceful handlers
- Implement handlers in `internal/app/api/<type>/handler/`
- Register created handlers in `internal/app/api/<type>/router.go`

Example handler structure:
```go
package handler

import (
	"net/http"
	membershipRepository "service/internal/membership/repository"
	"service/internal/pkg/form"
	"service/internal/pkg/parser"
	"service/internal/user/repository"
	"service/internal/user/service"

	xtremeres "github.com/globalxtreme/go-core/v2/response"
	"github.com/gorilla/mux"
)

type UserHandler struct{}

func (UserHandler) Get(w http.ResponseWriter, r *http.Request) {
	repo := repository.NewUserRepository()
	users, pagination, _ := repo.Paginate(r.URL.Query())

	psr := parser.UserParser{Array: users}
	res := xtremeres.Response{Array: psr.Briefs(), Pagination: &pagination}
	res.Success(w)
}

func (UserHandler) Detail(w http.ResponseWriter, r *http.Request) {
	repo := repository.NewUserRepository()
	user := repo.FirstById(mux.Vars(r)["id"], repo.PreloadMemberships)

	psr := parser.UserParser{Object: user}
	res := xtremeres.Response{Object: psr.First()}
	res.Success(w)
}

func (UserHandler) Create(w http.ResponseWriter, r *http.Request) {
	var form form.UserForm
	form.APIParse(r)
	form.Validate()

	srv := service.NewUserService()
	srv.SetMembershipRepository(membershipRepository.NewMembershipRepository())
	user := srv.Create(form)

	psr := parser.UserParser{Object: user}
	res := xtremeres.Response{Object: psr.First()}
	res.Success(w)
}

func (UserHandler) Update(w http.ResponseWriter, r *http.Request) {
	var form form.UserForm
	form.APIParse(r)
	form.Validate()

	srv := service.NewUserService()
	srv.SetMembershipRepository(membershipRepository.NewMembershipRepository())
	user := srv.Update(form, mux.Vars(r)["id"])

	psr := parser.UserParser{Object: user}
	res := xtremeres.Response{Object: psr.First()}
	res.Success(w)
}

func (UserHandler) Delete(w http.ResponseWriter, r *http.Request) {
	srv := service.NewUserService()
	srv.Delete(mux.Vars(r)["id"])

	res := xtremeres.Response{}
	res.Success(w)
}

func (UserHandler) ReportExcel(w http.ResponseWriter, r *http.Request) {
	var form form.UserReportForm
	form.APIParse(r)
	form.Validate()

	srv := service.NewUserService()
	filename := srv.ReportExcel(form)

	res := xtremeres.Response{Object: filename}
	res.Success(w)
}
```

Registered handlers in `internal/app/api/web/router.go` format:
```go
package web

import (
	"service/internal/app/api/web/handler"

	"github.com/gorilla/mux"
)

func Register(router *mux.Router) {
	//...other routes
	userRouter(router)
}

// ...other routers

func userRouter(router *mux.Router) {
	router = router.PathPrefix("/users").Subrouter()

	var userHandler handler.UserHandler
	router.HandleFunc("", userHandler.Get).Methods("GET")
	router.HandleFunc("", userHandler.Create).Methods("POST")
	router.HandleFunc("/{id}", userHandler.Detail).Methods("GET")
	router.HandleFunc("/{id}", userHandler.Update).Methods("PUT")
	router.HandleFunc("/{id}", userHandler.Delete).Methods("DELETE")
	router.HandleFunc("/download/excel", userHandler.ReportExcel).Methods("POST")
}
```

## Technology Stack

### Core Dependencies
- **GORM**: ORM for database operations
- **Gorilla Mux**: HTTP router
- **gRPC**: For service communication
- **Protocol Buffers**: Message serialization
- **RabbitMQ**: Message queuing with amqp091-go
- **Cobra**: CLI framework
- **Go-cache**: In-memory caching
- **Excelize**: Excel file processing
- **CORS**: Cross-origin resource sharing

### Custom Framework
- Uses `github.com/globalxtreme/go-core/v2` as base framework
- Leverage framework utilities for common operations

## File Generation Patterns

### When creating new features:
1. **Model**: Create in `internal/pkg/model/`
2. **Migration**: Use timestamp-based naming in `internal/app/database/migration/`
3. **Repository**: Interface in `port/repository.go`, implementation in domain folder
4. **Service**: Business logic in `internal/testing/service/` (adjust domain as needed)
5. **Forms**: Request validation in `internal/pkg/form/`
6. **Handlers**: API endpoints in `internal/app/api/*/handler/`
7. **Parsers**: Data transformation in `internal/pkg/parser/`

### File Naming Conventions
- Use PascalCase for Go files: `YourModelName.go`
- Include purpose in filename: `YourModelRepository.go`, `YourModelService.go`
- Use descriptive names for handlers: `YourFeatureHandler.go`

## Common Patterns

### Error Handling
- Use custom error types from `internal/pkg/error/`
- Create domain-specific errors
- Handle and log errors appropriately

### Configuration
- Configuration files in `internal/pkg/config/`
- Use environment-based configuration
- Follow existing patterns for new config types

### Constants
- Define constants in `internal/pkg/constant/`
- Group related constants in separate files
- Use descriptive constant names

### Queue Jobs
- Implement jobs in `internal/app/queue/job/`
- Follow existing job patterns
- Handle job failures gracefully

### gRPC Services
- Protocol buffers in `cmd/protobuf/`
- Generated code in `internal/pkg/grpc/`
- Server implementations in `internal/app/grpc/server/`

## Testing Patterns
- Create test files with `_test.go` suffix
- Use table-driven tests where appropriate
- Mock external dependencies
- Test business logic thoroughly

## CLI Commands
- Use Cobra for CLI commands
- Implement commands in `internal/app/console/command/`
- Follow existing command patterns

## Best Practices
1. Always validate input data using forms
2. Use proper error handling and logging
3. Follow the established repository pattern
4. Keep business logic in service layer
5. Use appropriate HTTP status codes
6. Implement proper transaction handling for database operations
7. Use constants for magic numbers and strings
8. Follow Go naming conventions
9. Write comprehensive tests
10. Document complex business logic

## When suggesting code:
- Follow the existing architecture patterns
- Use the established error handling mechanisms
- Leverage the custom framework utilities
- Maintain consistency with existing code style
- Consider performance and scalability
- Include proper validation and error handling
- Use appropriate logging where necessary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bypr1)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/bypr1)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
