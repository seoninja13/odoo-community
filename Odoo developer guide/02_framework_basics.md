# Odoo Framework Basics

# Creating Your First Odoo Module

## Introduction to Module Development
This chapter covers the foundation for creating a new Odoo module from scratch. We'll start with the minimum requirements to have a module recognized by Odoo.

## Real Estate Module Example
The example used throughout this guide is a Real Estate Advertisement module, chosen because:
- It represents a specific business case not covered by standard Odoo modules
- It demonstrates good practice of verifying that existing modules don't already cover the need
- It provides practical examples of common Odoo features

### Module Features Overview
The module will include:
- Property listings with detailed information
- Property characteristics (bedrooms, living area, garage, garden)
- Offer management system
- Custom workflow for property sales

## Module Directory Structure

### Minimum Required Files
A new Odoo module requires at least two files:
1. `__init__.py` - Python package identifier (can be empty initially)
2. `__manifest__.py` - Module descriptor file

### Manifest File Requirements
The `__manifest__.py` file must include:
- Required field: `name`
- Common additional information:
  - Category
  - Summary
  - Website
  - Dependencies (`depends` list)
  - Application status (`application` boolean)

### Dependencies Management
- Dependencies are listed in the `depends` list
- Odoo ensures dependent modules are installed first
- If a dependency is uninstalled, dependent modules are also uninstalled
- Similar to Linux package management systems

## Creating a New Module
1. Create the module directory structure:
   ```
   /path/to/addons/estate/
   ├── __init__.py
   └── __manifest__.py
   ```

2. Minimum manifest content:
   ```python
   {
       'name': 'Real Estate',
       'depends': ['base'],
       'application': True,
   }
   ```

3. Installation Steps:
   - Restart Odoo server
   - Enable developer mode
   - Go to Apps
   - Click "Update Apps List"
   - Search for your module
   - Install the module

## Next Steps
After creating the basic module structure:
- Create models for data structure
- Define views for user interface
- Implement business logic
- Add custom workflows

## Tips and Best Practices
- Always verify if existing modules can meet your needs before creating new ones
- Keep the initial module simple and build features incrementally
- Follow Odoo's naming conventions
- Document your module properly in the manifest
