# Chapter 10: Constraints

In Odoo, data consistency is crucial for maintaining a reliable business application. While we've added business logic through actions and buttons, we also need to prevent users from entering invalid data. This chapter explores two powerful mechanisms for enforcing data integrity: SQL constraints and Python constraints.

## SQL Constraints

SQL constraints are database-level restrictions that ensure data consistency directly at the database level. They are efficient and should be your first choice when implementing simple validation rules.

### Defining SQL Constraints

SQL constraints are defined through the `_sql_constraints` model attribute. The syntax is:

```python
_sql_constraints = [
    ('name', 'sql_definition', 'error_message'),
]
```

Example:
```python
class EstateProperty(models.Model):
    _name = 'estate.property'
    
    _sql_constraints = [
        ('check_expected_price', 'CHECK(expected_price > 0)',
         'The expected price must be strictly positive!'),
        ('check_selling_price', 'CHECK(selling_price >= 0)',
         'The selling price must be positive!')
    ]
```

### Common SQL Constraint Types

1. **CHECK Constraints**: Validate field values
   ```python
   ('check_price', 'CHECK(price > 0)', 'Price must be positive!')
   ```

2. **UNIQUE Constraints**: Ensure field uniqueness
   ```python
   ('unique_name', 'UNIQUE(name)', 'Name must be unique!')
   ```

3. **Multiple Field Constraints**: Combine multiple fields
   ```python
   ('unique_partner_ref', 'UNIQUE(partner_id, ref)', 
    'Partner reference must be unique per partner!')
   ```

## Python Constraints

When validation logic becomes more complex or requires Python computation, use Python constraints. These are defined using the `@api.constrains` decorator.

### Basic Structure

```python
from odoo import api, models
from odoo.exceptions import ValidationError

class EstateProperty(models.Model):
    _name = 'estate.property'

    @api.constrains('selling_price', 'expected_price')
    def _check_price_difference(self):
        for record in self:
            if not float_is_zero(record.selling_price, precision_digits=2):
                if float_compare(record.selling_price, record.expected_price * 0.9, precision_digits=2) < 0:
                    raise ValidationError(
                        "The selling price cannot be lower than 90% of the expected price!")
```

### Key Points About Python Constraints

1. **Field Dependencies**: List all fields that should trigger the constraint in the decorator
2. **Validation Timing**: Constraints are checked automatically when fields are modified
3. **Multiple Records**: Always loop through self as constraints can be called on recordsets
4. **Float Comparisons**: Use `float_compare` and `float_is_zero` for floating-point comparisons

## Best Practices

1. **Choose the Right Type**:
   - Use SQL constraints for simple field validations
   - Use Python constraints for complex business rules
   - SQL constraints are more efficient than Python constraints

2. **Error Messages**:
   - Make error messages clear and actionable
   - Include specific information about why the validation failed
   - Consider internationalization (i18n) for error messages

3. **Performance Considerations**:
   - SQL constraints are checked at the database level
   - Python constraints require loading records into memory
   - Multiple constraints can impact save operations

4. **Testing**:
   - Test both positive and negative cases
   - Verify constraint behavior with edge cases
   - Check constraint interaction with existing data

## Exercises

1. Add SQL constraints to the estate module:
   - Ensure expected_price is strictly positive
   - Ensure selling_price is positive
   - Make property type names unique
   - Make property tag names unique

2. Add Python constraints:
   - Prevent selling price from being lower than 90% of expected price
   - Handle the case where selling price is zero (not yet sold)
   - Use proper float comparison utilities

3. Test your constraints:
   - Try creating records with invalid data
   - Verify error messages are clear
   - Test updating existing records

## Common Pitfalls

1. **Float Comparison**:
   ```python
   # Wrong
   if price1 == price2:  # Don't compare floats directly
   
   # Correct
   from odoo.tools.float_utils import float_compare
   if float_compare(price1, price2, precision_digits=2) == 0:
   ```

2. **Missing Dependencies**:
   ```python
   # Wrong - missing 'price' in decorator
   @api.constrains('name')
   def _check_price(self):
       self.ensure_one()
       if self.price < 0:
           raise ValidationError("Price must be positive")
   
   # Correct
   @api.constrains('name', 'price')
   def _check_price(self):
       self.ensure_one()
       if self.price < 0:
           raise ValidationError("Price must be positive")
   ```

3. **Inefficient Constraints**:
   ```python
   # Inefficient - loads all records
   @api.constrains('name')
   def _check_unique_name(self):
       all_records = self.search([])
       # ... check uniqueness
   
   # Better - use SQL constraint
   _sql_constraints = [
       ('unique_name', 'UNIQUE(name)', 'Name must be unique!')
   ]
   ```

Remember that constraints are crucial for maintaining data integrity in your Odoo applications. Choose the appropriate constraint type based on your needs, and always consider performance implications when implementing them.
