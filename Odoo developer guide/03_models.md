# Odoo Models and ORM

## Introduction to Models
Models in Odoo are the foundation for storing data in the database. They define the structure and behavior of business objects through Python classes.

## Important Warning
**Do not use mutable global variables**
- A single Odoo instance can run multiple databases
- Different modules might be installed on each database
- Global variables cannot be relied upon for module-specific state

## Object-Relational Mapping (ORM)
The ORM layer provides:
- Automated SQL generation
- Extensibility
- Security services
- Database persistence

### Basic Model Structure
```python
from odoo import models

class TestModel(models.Model):
    _name = "test_model"
    _description = "Test Model"
```

### Model Organization
- Models are typically located in a `models` directory
- Each model is defined in its own Python file
- Files must be properly imported through `__init__.py` files

## Field Types

### Simple Fields
1. **Basic Types**
   - `Char`: String fields (VARCHAR)
   - `Text`: Long text fields
   - `Integer`: Whole numbers
   - `Float`: Decimal numbers
   - `Boolean`: True/False values
   - `Date`: Date fields
   - `Selection`: Predefined list of options

2. **Example Field Definitions**
```python
from odoo import fields, models

class RealEstateProperty(models.Model):
    _name = "estate.property"
    _description = "Real Estate Property"

    name = fields.Char(required=True)
    description = fields.Text()
    postcode = fields.Char()
    date_availability = fields.Date()
    expected_price = fields.Float(required=True)
    selling_price = fields.Float()
    bedrooms = fields.Integer()
    living_area = fields.Integer()
    facades = fields.Integer()
    garage = fields.Boolean()
    garden = fields.Boolean()
    garden_area = fields.Integer()
    garden_orientation = fields.Selection([
        ('north', 'North'),
        ('south', 'South'),
        ('east', 'East'),
        ('west', 'West')
    ])
```

## Field Attributes

### Common Attributes
1. **Basic Attributes**
   - `string`: Field label in UI
   - `required`: Makes field mandatory
   - `help`: Tooltip text
   - `index`: Creates database index

2. **Usage Example**
```python
name = fields.Char(
    string="Property Name",
    required=True,
    help="Enter the name of the property",
    index=True
)
```

## Automatic Fields
Fields automatically created by Odoo:
- `id`: Unique identifier
- `create_date`: Creation timestamp
- `create_uid`: Creator user ID
- `write_date`: Last modification timestamp
- `write_uid`: Last modifier user ID

### Characteristics
- System-managed
- Read-only
- Available in all models
- Can be disabled if needed

## Database Management
1. **Table Creation**
   - Tables are automatically created from model definitions
   - Follow naming conventions (e.g., `estate_property` for `estate.property`)

2. **Schema Updates**
   - Require server restart with specific parameters
   - Use `-u module_name` for updates
   - Use `-d database_name` to specify database

3. **Example Command**
```bash
./odoo-bin --addons-path=addons,../enterprise/,../tutorials/ -d database_name -u module_name
```

## Computed Fields and Onchange Methods

### Computed Fields
```python
from odoo import api, fields, models

class RealEstateProperty(models.Model):
    _name = "estate.property"
    
    # Computed field example
    total_area = fields.Float(compute="_compute_total_area")
    
    @api.depends('living_area', 'garden_area')
    def _compute_total_area(self):
        for record in self:
            record.total_area = record.living_area + record.garden_area
```

Key Features:
1. Always use `@api.depends` decorator
2. List all fields that affect computation
3. Iterate through records
4. Can be stored in database with `store=True`

### Onchange Methods
```python
from odoo import api, fields, models

class RealEstateProperty(models.Model):
    _name = "estate.property"
    
    garden = fields.Boolean()
    garden_area = fields.Integer()
    garden_orientation = fields.Selection([
        ('north', 'North'),
        ('south', 'South'),
        ('east', 'East'),
        ('west', 'West')
    ])
    
    @api.onchange('garden')
    def _onchange_garden(self):
        if self.garden:
            self.garden_area = 10
            self.garden_orientation = 'north'
        else:
            self.garden_area = 0
            self.garden_orientation = False
```

Key Points:
1. Only triggered in form views
2. Used for UI/UX improvements
3. Cannot contain business logic
4. Works on single record (`self`)

### Warning Messages in Onchange
```python
@api.onchange('partner_id')
def _onchange_partner(self):
    if self.partner_id:
        return {
            'warning': {
                'title': 'Warning',
                'message': 'Changing the partner will update related fields.'
            }
        }
```

## Choosing Between Computed Fields and Onchange

### When to Use Computed Fields
1. Business logic implementation
2. Values needed outside form views
3. Consistent data across UI
4. Complex calculations

Example:
```python
total_price = fields.Float(
    compute='_compute_total_price',
    store=True
)

@api.depends('base_price', 'tax_rate')
def _compute_total_price(self):
    for record in self:
        record.total_price = record.base_price * (1 + record.tax_rate)
```

### When to Use Onchange
1. Form view helpers
2. Default value suggestions
3. User experience improvements
4. Non-critical field updates

Example:
```python
@api.onchange('type_id')
def _onchange_type(self):
    if self.type_id:
        self.name = self.type_id.default_name
        self.description = self.type_id.default_description
```

### Best Practices

1. Computed Fields
   - Always specify dependencies
   - Consider performance impact
   - Use storage wisely
   - Document complex computations

2. Onchange Methods
   - Keep logic simple
   - Focus on UX
   - Avoid business rules
   - Use clear warning messages

3. Performance Considerations
   - Watch computed field dependencies
   - Avoid chained computations
   - Consider caching for complex calculations
   - Test with large datasets

### Common Pitfalls

1. Computed Fields
   - Missing dependencies
   - Circular dependencies
   - Unnecessary storage
   - Performance issues with large datasets

2. Onchange Methods
   - Complex business logic
   - Dependency on external data
   - Unclear field updates
   - Confusing user experience

### Debugging Tips

1. Computed Fields
   - Check dependency triggers
   - Monitor recomputation logs
   - Test edge cases
   - Verify stored values

2. Onchange Methods
   - Use developer tools
   - Check field updates
   - Test user scenarios
   - Monitor warning messages

## Best Practices
1. **Model Organization**
   - Keep models in dedicated files
   - Use proper import structure
   - Follow naming conventions

2. **Field Definition**
   - Choose appropriate field types
   - Set meaningful descriptions
   - Consider indexing for search-heavy fields

3. **Security Considerations**
   - Always define access rules
   - Consider record rules
   - Handle sensitive data appropriately
