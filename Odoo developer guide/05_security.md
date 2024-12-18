# Odoo Security

## Introduction to Security
Security in Odoo determines who can access what data. It's a crucial aspect that should be considered early in module development.

## Data Files Management

### CSV Data Files
- Used for loading configuration data
- Located in specific folders:
  - `data/`: General data files
  - `security/`: Security-related files
  - `views/`: UI-related files
- Must be declared in `__manifest__.py`
- Loaded only during module installation/update

### CSV File Structure
Example of country states data:
```csv
"id","country_id:id","name","code"
state_au_1,au,"Australian Capital Territory","ACT"
state_au_2,au,"New South Wales","NSW"
```

Key Components:
1. `id`: External identifier
2. References to other records: `field_name:id`
3. Regular data fields

### Important Considerations
- Files are loaded sequentially as listed in `__manifest__.py`
- Dependencies must be loaded in correct order
- Data referring to other data must be loaded after its dependencies

## Access Rights

### Basic Concepts
- Defined in `ir.model.access` records
- Controls CRUD operations (Create, Read, Update, Delete)
- Can be global or group-specific
- Typically defined in `ir.model.access.csv`

### Access Rights Structure
```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_test_model,access_test_model,model_test_model,base.group_user,1,0,0,0
```

Fields Explanation:
- `id`: Unique identifier
- `name`: Description
- `model_id/id`: Referenced model (format: `model_<model_name>`)
- `group_id/id`: User group
- `perm_read`: Read permission (0/1)
- `perm_write`: Write permission (0/1)
- `perm_create`: Create permission (0/1)
- `perm_unlink`: Delete permission (0/1)

### Implementation Example
1. Create security file structure:
```
your_module/
├── security/
│   └── ir.model.access.csv
```

2. Define access rights in CSV:
```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_estate_property,access_estate_property,model_estate_property,base.group_user,1,1,1,1
```

3. Add to `__manifest__.py`:
```python
{
    'name': 'Real Estate',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
    ],
}
```

## Best Practices

### File Organization
1. Keep security files in dedicated `security/` folder
2. Use clear, consistent naming conventions
3. Document access rights decisions

### Access Rights Management
1. Start with minimal required access
2. Consider security implications before granting permissions
3. Use specific groups instead of global access when possible
4. Regular audit of access rights

### Common Pitfalls
1. Forgetting to declare files in manifest
2. Incorrect model references
3. Wrong loading order
4. Overly permissive access rights

## Troubleshooting

### Common Warnings
```
WARNING odoo.modules.loading: The models ['model.name'] have no access rules...
```
Solution: Create appropriate access rights in `ir.model.access.csv`

### Security Checks
1. Verify file declarations in manifest
2. Check model references
3. Confirm group existence
4. Test with different user roles
