# ECommerce Solution

A modern e-commerce application built with .NET, following Clean Architecture principles for maintainability, testability, and scalability.

## 🏗️ Architecture

This project implements **Clean Architecture** with clear separation of concerns across multiple layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│  ┌─────────────────┐              ┌─────────────────┐       │
│  │  ECommerce.API  │              │  ECommerce.Web  │       │
│  │   (REST API)    │              │   (MVC Web App) │       │
│  └─────────────────┘              └─────────────────┘       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                       │
│                   ┌─────────────────┐                       │
│                   │ ECommerce.Core  │                       │
│                   │ (Business Logic)│                       │
│                   └─────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                       │
│                ┌────────────────────────┐                   │
│                │ECommerce.Infrastructure│                   │
│                │(Data Access & External)│                   │
│                └────────────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Project Structure

### **ECommerce.Core** (Application Layer)
- **Entities**: Domain models and business objects
- **DTOs**: Data Transfer Objects for API communication
- **Interfaces**: Contracts for services and repositories
- **Services**: Business logic and application services

### **ECommerce.API** (Presentation Layer)
- **Controllers**: REST API endpoints
- **Middleware**: Custom middleware components
- RESTful API for mobile apps and external integrations

### **ECommerce.Web** (Presentation Layer)
- **Controllers**: MVC controllers for web interface
- **Views**: Razor pages and templates
- **ViewModels**: Models for view binding
- **Services**: UI-specific services
- Web-based user interface

### **ECommerce.Infrastructure** (Infrastructure Layer)
- **Data**: Database context and configurations
- **Repositories**: Data access implementations
- **Identity**: Authentication and authorization
- **Services**: External service integrations
- **Migrations**: Database schema changes

### **ECommerce.Tests** (Test Layer)
- Unit tests and integration tests
- Ensures code quality and reliability

## 🚀 Getting Started

### Prerequisites
- .NET 8.0 SDK or later
- SQL Server (LocalDB or full instance)
- Visual Studio 2022 or VS Code

### Quick Start
1. Clone the repository
2. Restore NuGet packages
3. Update connection strings in `appsettings.json`
4. Run database migrations
5. Start the application

## 🛠️ Technology Stack

- **Framework**: .NET 8
- **Architecture**: Clean Architecture
- **Database**: SQL Server with Entity Framework Core
- **Authentication**: ASP.NET Core Identity
- **API**: ASP.NET Core Web API
- **Frontend**: ASP.NET Core MVC with Razor Pages
- **Testing**: xUnit

## 📋 Features

- User authentication and authorization
- Product catalog management
- Shopping cart functionality
- Order processing
- RESTful API endpoints
- Responsive web interface
- Clean separation of concerns
- Comprehensive testing suite

## 🤝 Contributing

This project follows Clean Architecture principles. Please ensure:
- Business logic stays in the Core layer
- Dependencies point inward
- Infrastructure concerns remain in the Infrastructure layer
- Proper unit test coverage

## 📄 License

This project is licensed under the MIT License.
