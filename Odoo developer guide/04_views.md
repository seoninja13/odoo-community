# Views and UI Elements in Odoo

## Data Files in XML Format

### Introduction to XML Data Files
- Used for complex data structures
- More flexible than CSV for UI elements
- Stored in `views/` directory
- Must be declared in `__manifest__.py`
- Loaded sequentially during module installation/update

### Basic XML Structure
```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Content goes here -->
    </data>
</odoo>
```

## Actions

### Window Actions (ir.actions.act_window)
```xml
<record id="estate_property_action" model="ir.actions.act_window">
    <field name="name">Properties</field>
    <field name="res_model">estate.property</field>
    <field name="view_mode">tree,form,kanban</field>
</record>
```

Key Components:
- `id`: External identifier
- `model`: Always `ir.actions.act_window`
- `name`: Action display name
- `res_model`: Target model
- `view_mode`: Available view types

### Action Triggers
1. Menu items
2. Button clicks
3. Contextual actions

## Menu Structure

### Menu Hierarchy
1. Root Menu (App Switcher)
2. First Level Menu (Top Bar)
3. Action Menus (Submenus)

### Menu Definition
```xml
<!-- Root Menu -->
<menuitem 
    id="estate_menu_root"
    name="Real Estate"
    sequence="10"/>

<!-- First Level Menu -->
<menuitem 
    id="estate_first_level_menu"
    name="Properties"
    parent="estate_menu_root"
    sequence="10"/>

<!-- Action Menu -->
<menuitem 
    id="estate_property_menu_action"
    name="Properties"
    parent="estate_first_level_menu"
    action="estate_property_action"
    sequence="10"/>
```

## Views

### Basic View Types

#### List View (Tree View)
```xml
<record id="estate_property_tree" model="ir.ui.view">
    <field name="name">estate.property.tree</field>
    <field name="model">estate.property</field>
    <field name="arch" type="xml">
        <tree>
            <field name="name"/>
            <field name="postcode"/>
            <field name="bedrooms"/>
            <field name="living_area"/>
            <field name="expected_price"/>
            <field name="selling_price"/>
            <field name="date_availability"/>
            <field name="state"/>
        </tree>
    </field>
</record>
```

Key Features:
- Tabular display of records
- Column-based layout
- Optional editable mode
- Sortable columns

#### Form View
```xml
<record id="estate_property_form" model="ir.ui.view">
    <field name="name">estate.property.form</field>
    <field name="model">estate.property</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                        <field name="postcode"/>
                        <field name="date_availability"/>
                    </group>
                    <group>
                        <field name="expected_price"/>
                        <field name="selling_price"/>
                        <field name="state"/>
                    </group>
                </group>
                <notebook>
                    <page string="Description">
                        <group>
                            <field name="description"/>
                            <field name="bedrooms"/>
                            <field name="living_area"/>
                            <field name="facades"/>
                            <field name="garage"/>
                            <field name="garden"/>
                            <field name="garden_area"/>
                            <field name="garden_orientation"/>
                        </group>
                    </page>
                </notebook>
            </sheet>
        </form>
    </field>
</record>
```

Structure Elements:
1. `<sheet>`: Main content container
2. `<group>`: Field organization
3. `<notebook>`: Tab container
4. `<page>`: Individual tabs

#### Search View
```xml
<record id="estate_property_search" model="ir.ui.view">
    <field name="name">estate.property.search</field>
    <field name="model">estate.property</field>
    <field name="arch" type="xml">
        <search>
            <field name="name"/>
            <field name="postcode"/>
            <field name="expected_price"/>
            <field name="bedrooms"/>
            <field name="living_area"/>
            <field name="facades"/>
            
            <filter string="Available" 
                    name="available" 
                    domain="['|', ('state', '=', 'new'), 
                               ('state', '=', 'offer_received')]"/>
            
            <group expand="1" string="Group By">
                <filter string="Postcode" 
                        name="postcode" 
                        context="{'group_by': 'postcode'}"/>
            </group>
        </search>
    </field>
</record>
```

Components:
1. Search Fields
2. Filters
3. Group By options

## Relational Views

### One2many Field Views
When displaying related records in a form view, you'll typically use a one2many field. Here's an example with property offers:

```xml
<!-- Property Form View with Offers -->
<record id="estate_property_form" model="ir.ui.view">
    <field name="name">estate.property.form</field>
    <field name="model">estate.property</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <!-- Other property fields... -->
                <notebook>
                    <page string="Offers">
                        <field name="offer_ids">
                            <tree>
                                <field name="price"/>
                                <field name="partner_id"/>
                                <field name="status"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
        </form>
    </field>
</record>
```

### Embedded Views
For related records that are accessed only through their parent:
```xml
<!-- Offer List View (Embedded) -->
<field name="offer_ids">
    <tree editable="bottom">
        <field name="price"/>
        <field name="partner_id"/>
        <field name="status"/>
    </tree>
</field>
```

Key Points:
1. No separate menu needed for child records
2. Required fields may be omitted if automatically filled
3. Parent-child relationship handled by framework
4. Editable lists for quick data entry

### Form View Inside One2many
You can also define a form view for detailed editing:
```xml
<field name="offer_ids">
    <form>
        <group>
            <field name="price"/>
            <field name="partner_id"/>
            <field name="status"/>
        </group>
    </form>
</field>
```

### Implicit Field Handling
- Many2one fields (like `property_id`) can be omitted in child views
- Framework automatically sets the parent reference
- Reduces redundant data entry
- Maintains data integrity

### Best Practices for Relational Views

1. View Organization
   - Keep related fields grouped
   - Use appropriate view types
   - Consider user workflow

2. Field Selection
   - Show relevant fields only
   - Omit automatically filled fields
   - Include status and key information

3. Performance Considerations
   - Use `editable="bottom"` for quick entry
   - Consider pagination for large datasets
   - Lazy load related records

4. User Experience
   - Provide clear labels
   - Group related actions
   - Enable inline editing when appropriate

### Common Patterns

1. Master-Detail Views
```xml
<form>
    <sheet>
        <group>
            <!-- Master record fields -->
        </group>
        <notebook>
            <page string="Details">
                <!-- One2many field with embedded views -->
            </page>
        </notebook>
    </sheet>
</form>
```

2. Quick Entry Lists
```xml
<field name="related_ids">
    <tree editable="bottom" create="true" delete="true">
        <!-- Essential fields for quick entry -->
    </tree>
</field>
```

3. Detailed Forms
```xml
<field name="related_ids">
    <form>
        <group>
            <!-- All available fields for detailed editing -->
        </group>
    </form>
</field>
```

### Troubleshooting

1. Common Issues
   - Missing required fields
   - Incorrect field references
   - Permission problems
   - View inheritance conflicts

2. Solutions
   - Check field definitions
   - Verify security settings
   - Test with minimal views
   - Debug view inheritance

3. Development Tips
   - Use developer mode
   - Check server logs
   - Test with sample data
   - Validate XML syntax

## Domains

### Domain Syntax
Domains are used to filter records using conditions:
```python
[
    ('field_name', 'operator', value),
    '|',  # OR operator
    ('field_1', '=', 'value_1'),
    ('field_2', '!=', 'value_2'),
]
```

### Common Operators
- `=`, `!=`: Equality, inequality
- `>`, `>=`, `<`, `<=`: Comparison
- `in`, `not in`: List membership
- `like`, `ilike`: Pattern matching
- `child_of`: Hierarchical relationship

### Logical Operators
- `&`: AND (default)
- `|`: OR
- `!`: NOT

### XML Domain Examples
```xml
<!-- Available properties filter -->
<filter name="available" 
        domain="['|', 
                ('state', '=', 'new'),
                ('state', '=', 'offer_received')]"/>

<!-- Price range filter -->
<filter name="high_value" 
        domain="[('expected_price', '>=', 100000)]"/>
```

## Field Attributes and Views

### Common Field Attributes
```python
name = fields.Char(
    string="Property Name",
    required=True,
    readonly=True,
    copy=False,
    default="New Property"
)
```

1. Basic Attributes:
   - `string`: Label in UI
   - `required`: Mandatory field
   - `readonly`: Non-editable
   - `copy`: Copy behavior
   - `default`: Default value

2. Default Values:
   ```python
   # Static default
   bedrooms = fields.Integer(default=2)
   
   # Dynamic default
   date_availability = fields.Date(
       default=lambda self: fields.Date.today() + relativedelta(months=3)
   )
   ```

### Reserved Fields
1. `active` Field:
   ```python
   active = fields.Boolean(default=True)
   ```
   - Controls record visibility
   - Inactive records hidden by default

2. `state` Field:
   ```python
   state = fields.Selection([
       ('new', 'New'),
       ('offer_received', 'Offer Received'),
       ('offer_accepted', 'Offer Accepted'),
       ('sold', 'Sold'),
       ('cancelled', 'Cancelled')
   ], required=True, copy=False, default='new')
   ```
   - Manages record lifecycle
   - Controls UI states

## File Organization

### Directory Structure
```
your_module/
├── views/
│   ├── estate_property_views.xml
│   └── estate_menus.xml
```

### Manifest Declaration
```python
{
    'name': 'Real Estate',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/estate_property_views.xml',
        'views/estate_menus.xml',
    ],
}
```

## Best Practices

### XML Files
1. Organize by functionality
2. Use meaningful IDs
3. Follow loading order
4. Comment complex structures

### Field Attributes
1. Use meaningful defaults
2. Consider copy behavior
3. Set appropriate readonly states
4. Document field purposes

### Menu Organization
1. Logical grouping
2. Consistent naming
3. Appropriate sequencing
4. Clear hierarchy

### View Organization
1. Logical field grouping
2. Intuitive tab structure
3. Consistent naming conventions
4. Clear filter definitions

### Search View Design
1. Include commonly searched fields
2. Provide useful filters
3. Add relevant grouping options
4. Consider user workflow

### Domain Construction
1. Use proper operator precedence
2. Escape special characters in XML
3. Test complex domains
4. Document complex filters

## Development Tips

### Server Parameters
```bash
./odoo-bin --addons-path=addons,../enterprise/,../tutorials/ -d database -u module --dev xml
```
- `--dev xml`: Reload views without server restart
- `-u module`: Update specific module
- `-d database`: Specify database

### Common Issues

#### View Not Updating
- Clear browser cache
- Check XML syntax
- Verify view inheritance

#### Domain Problems
- Validate operator syntax
- Check value types
- Test simpler versions first

#### Search View Issues
- Verify field names
- Check domain syntax
- Test filter combinations
