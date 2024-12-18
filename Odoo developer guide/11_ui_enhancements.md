# Chapter 11: UI Enhancements

This chapter covers various techniques to enhance the user interface of your Odoo modules. We'll explore how to make views more dynamic, visually appealing, and user-friendly.

## Inline Views

Inline views allow you to customize how related records are displayed within a form view. Instead of using the default view, you can specify exactly which fields to show and how to show them.

### Basic Structure

```xml
<field name="line_ids">
    <tree>
        <field name="name"/>
        <field name="expected_price"/>
        <field name="state"/>
    </tree>
</field>
```

### Example: Property Type Form View
```xml
<form>
    <sheet>
        <group>
            <field name="name"/>
            <field name="property_ids">
                <tree>
                    <field name="name"/>
                    <field name="expected_price"/>
                    <field name="state"/>
                </tree>
            </field>
        </group>
    </sheet>
</form>
```

## Widgets

Widgets provide specialized ways to display and interact with fields. Odoo offers various built-in widgets for different field types.

### Common Widgets

1. **Statusbar**:
```xml
<field name="state" widget="statusbar" 
       statusbar_visible="new,offer_received,offer_accepted,sold"/>
```

2. **Many2many Tags**:
```xml
<field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
```

3. **Handle** (for manual sorting):
```xml
<field name="sequence" widget="handle"/>
```

## List View Enhancements

### Editable Lists
Make lists directly editable without opening form views:
```xml
<tree editable="bottom">
    <field name="name"/>
    <field name="price"/>
</tree>
```

### Decorations
Add visual indicators based on record states:
```xml
<tree decoration-success="state == 'offer_accepted'"
      decoration-danger="state == 'canceled'"
      decoration-muted="state == 'sold'">
    <field name="name"/>
    <field name="state"/>
</tree>
```

### Optional Fields
Make fields optionally visible in list views:
```xml
<field name="date_availability" optional="hide"/>
```

## Form View Attributes

### Conditional Display
Control field visibility and editability based on conditions:

```xml
<!-- Hide field based on condition -->
<field name="garden_area" invisible="not garden"/>

<!-- Make field readonly based on state -->
<field name="offer_ids" readonly="state in ('offer_accepted','sold','canceled')"/>

<!-- Required field based on condition -->
<field name="buyer_id" required="state == 'offer_accepted'"/>
```

### Button States
Control button visibility based on state:
```xml
<button name="action_sold" 
        string="Sold" 
        type="object"
        invisible="state != 'offer_accepted'"/>
```

## Search View Improvements

### Default Filters
Set default filters in the action:
```python
{
    'name': 'Properties',
    'res_model': 'estate.property',
    'type': 'ir.actions.act_window',
    'context': {'search_default_available': 1},
}
```

### Custom Search Domains
Customize how search fields work:
```xml
<field name="living_area" 
       filter_domain="[('living_area', '>=', self)]"/>
```

## Stat Buttons

Stat buttons provide quick access to related information with counters:

```xml
<button name="%(action_property_offers)d"
        type="action"
        class="oe_stat_button"
        icon="fa-money">
    <field name="offer_count" string="Offers" widget="statinfo"/>
</button>
```

### Implementation Example

1. Add related field:
```python
class PropertyOffer(models.Model):
    _name = 'estate.property.offer'
    
    property_type_id = fields.Many2one(
        related='property_id.property_type_id',
        store=True)
```

2. Add computed counter:
```python
class PropertyType(models.Model):
    _name = 'estate.property.type'
    
    offer_count = fields.Integer(
        compute='_compute_offer_count')
        
    def _compute_offer_count(self):
        for record in self:
            record.offer_count = len(record.offer_ids)
```

## Best Practices

1. **View Organization**:
   - Group related fields logically
   - Use appropriate widgets for better UX
   - Keep views clean and uncluttered

2. **Performance**:
   - Use stored related fields when needed
   - Avoid complex domains in views
   - Consider the impact of computed fields

3. **User Experience**:
   - Make common actions easily accessible
   - Use consistent styling and layout
   - Provide clear visual feedback

4. **Maintenance**:
   - Document view customizations
   - Keep view definitions modular
   - Follow Odoo naming conventions

## Exercises

1. Enhance Property Type View:
   - Add inline property list
   - Implement manual sorting
   - Add stat button for offers

2. Improve Property List View:
   - Add state-based colors
   - Make certain fields optional
   - Implement custom sorting

3. Optimize Search View:
   - Add default filters
   - Implement custom search domains
   - Add grouped searching options

Remember that UI enhancements should always serve a purpose and improve the user experience. Avoid adding decorative elements that don't provide value to the users.
