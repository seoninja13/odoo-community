# Chapter 14: QWeb Templates

QWeb is Odoo's primary templating engine used for generating HTML content. This chapter explores QWeb's capabilities through the implementation of Kanban views, one of its most common use cases.

## QWeb Basics

QWeb is an XML-based templating engine that provides:
- XML template definitions
- Dynamic content rendering
- Conditional rendering
- Loops and iterations
- Expression evaluation

### Basic Structure

```xml
<templates>
    <t t-name="template-name">
        <div>
            <span t-esc="value"/>
        </div>
    </t>
</templates>
```

## Kanban Views

### Basic Structure

```xml
<record id="view_estate_property_kanban" model="ir.ui.view">
    <field name="name">estate.property.kanban</field>
    <field name="model">estate.property</field>
    <field name="arch" type="xml">
        <kanban>
            <templates>
                <t t-name="kanban-box">
                    <div class="oe_kanban_card">
                        <field name="name"/>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
```

### Field Declaration

```xml
<kanban>
    <!-- Fields needed but not displayed -->
    <field name="state"/>
    <field name="tag_ids"/>
    
    <templates>
        <t t-name="kanban-box">
            <!-- Template content -->
        </t>
    </templates>
</kanban>
```

## QWeb Directives

### 1. Conditional Rendering (t-if)

```xml
<div t-if="record.state.raw_value == 'new'" 
     class="badge badge-success">
    New
</div>
```

### 2. Value Display (t-esc)

```xml
<span t-esc="record.expected_price.value"/>
```

### 3. Raw HTML (t-raw)

```xml
<div t-raw="record.description.value"/>
```

### 4. Loops (t-foreach)

```xml
<t t-foreach="record.tag_ids.raw_value" t-as="tag">
    <span class="badge badge-pill">
        <t t-esc="tag"/>
    </span>
</t>
```

## Complete Kanban Example

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_estate_property_kanban" model="ir.ui.view">
        <field name="name">estate.property.kanban</field>
        <field name="model">estate.property</field>
        <field name="arch" type="xml">
            <kanban default_group_by="property_type_id" 
                    default_order="create_date desc"
                    records_draggable="false">
                
                <!-- Declare fields -->
                <field name="name"/>
                <field name="state"/>
                <field name="expected_price"/>
                <field name="best_price"/>
                <field name="selling_price"/>
                <field name="tag_ids"/>
                <field name="property_type_id"/>
                
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <!-- Header -->
                            <div class="oe_kanban_details">
                                <strong class="o_kanban_record_title">
                                    <field name="name"/>
                                </strong>
                                
                                <!-- Tags -->
                                <div class="o_kanban_tags_section">
                                    <field name="tag_ids" widget="many2many_tags"/>
                                </div>
                                
                                <!-- Prices -->
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <span>Expected: 
                                            <field name="expected_price"/>
                                        </span>
                                    </div>
                                    
                                    <!-- Best Price -->
                                    <div t-if="record.best_price.raw_value > 0" 
                                         class="oe_kanban_bottom_right">
                                        <span>Best Offer: 
                                            <field name="best_price"/>
                                        </span>
                                    </div>
                                    
                                    <!-- Selling Price -->
                                    <div t-if="record.state.raw_value == 'offer_accepted'" 
                                         class="oe_kanban_bottom_right">
                                        <span>Selling Price: 
                                            <field name="selling_price"/>
                                        </span>
                                    </div>
                                </div>
                                
                                <!-- State Badge -->
                                <div class="o_kanban_record_top">
                                    <div t-attf-class="badge badge-#{record.state.raw_value}">
                                        <field name="state"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>
```

## Best Practices

1. **Template Organization**:
   - Keep templates focused and single-purpose
   - Use meaningful template names
   - Group related templates together

2. **Performance**:
   - Declare only needed fields
   - Use raw_value when possible
   - Minimize complex computations

3. **Styling**:
   - Use Odoo's built-in classes
   - Follow Bootstrap conventions
   - Keep styling consistent

## Common Patterns

### 1. Clickable Cards

```xml
<div class="oe_kanban_global_click">
    <!-- Card content -->
</div>
```

### 2. Status Indicators

```xml
<div t-attf-class="badge badge-#{record.state.raw_value}">
    <field name="state"/>
</div>
```

### 3. Grouped Layout

```xml
<kanban default_group_by="property_type_id">
    <!-- Kanban content -->
</kanban>
```

## Exercises

1. Basic Kanban View:
   - Create minimal structure
   - Add basic fields
   - Test rendering

2. Enhanced Display:
   - Add conditional elements
   - Include tags
   - Style the cards

3. Advanced Features:
   - Implement grouping
   - Add custom colors
   - Handle state changes

## Debugging Tips

1. **Template Issues**:
   - Check field declarations
   - Verify XML syntax
   - Test conditional logic

2. **Styling Problems**:
   - Inspect element classes
   - Check CSS hierarchy
   - Verify Bootstrap classes

3. **Data Display**:
   - Use raw_value vs value
   - Check field availability
   - Verify record access

Remember that QWeb templates are powerful but require attention to detail. Start simple and build complexity gradually.
