# Chapter 13: Module Interaction

One of Odoo's strengths is its modular architecture. This chapter explores how to create interactions between different modules while maintaining modularity and flexibility.

## Link Modules

### Purpose and Structure

Link modules serve as bridges between independent modules, allowing them to work together while maintaining modularity. They typically:
- Depend on both modules they connect
- Contain only the interaction logic
- Are optional for both base modules

### Example Structure

```
estate_account/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── estate_property.py
```

### Manifest File
```python
{
    'name': 'Real Estate Accounting',
    'description': 'Bridge module for real estate and accounting',
    'depends': [
        'estate',
        'account',
    ],
    'data': [],
    'application': False,
    'auto_install': True,
}
```

## Model Inheritance for Integration

### Basic Structure

```python
from odoo import models, Command

class EstateProperty(models.Model):
    _inherit = 'estate.property'

    def action_sold(self):
        # Add custom logic before or after the original method
        return super().action_sold()
```

### Full Implementation Example

```python
from odoo import models, Command

class EstateProperty(models.Model):
    _inherit = 'estate.property'

    def action_sold(self):
        # Create invoice before calling super
        self._create_invoice()
        return super().action_sold()

    def _create_invoice(self):
        for record in self:
            # Create invoice
            invoice_vals = {
                'partner_id': record.buyer_id.id,
                'move_type': 'out_invoice',
                'journal_id': self.env['account.journal'].search(
                    [('type', '=', 'sale')], limit=1).id,
                'invoice_line_ids': [
                    # 6% of selling price
                    Command.create({
                        'name': f'Property: {record.name}',
                        'quantity': 1.0,
                        'price_unit': record.selling_price * 0.06,
                    }),
                    # Administrative fees
                    Command.create({
                        'name': 'Administrative fees',
                        'quantity': 1.0,
                        'price_unit': 100.00,
                    }),
                ],
            }
            self.env['account.move'].create(invoice_vals)
```

## Working with Related Models

### Account Move (Invoice) Structure

Key fields for invoice creation:
```python
invoice_vals = {
    'partner_id': partner.id,      # Mandatory: customer
    'move_type': 'out_invoice',    # Type: customer invoice
    'journal_id': journal.id,      # Accounting journal
    'invoice_line_ids': [          # Invoice lines
        Command.create({
            'name': description,
            'quantity': quantity,
            'price_unit': price,
        }),
    ],
}
```

### Command Namespace

The `Command` namespace provides methods for handling O2M and M2M fields:

```python
from odoo import Command

# Create and link a new record
Command.create({values})

# Link an existing record
Command.link(id)

# Unlink a record
Command.unlink(id)

# Remove all records
Command.clear()

# Update a linked record
Command.update(id, values)
```

## Best Practices

1. **Modularity**:
   ```python
   # Good: Check if account module is installed
   if 'account.move' in self.env:
       self._create_invoice()
   ```

2. **Error Handling**:
   ```python
   def _create_invoice(self):
       self.ensure_one()
       if not self.buyer_id:
           raise UserError(_("Cannot create invoice without a buyer."))
   ```

3. **Clean Dependencies**:
   ```python
   # In __manifest__.py
   {
       'depends': [
           'estate',  # Base module
           'account', # Required for integration
       ],
   }
   ```

## Common Patterns

### 1. Pre/Post Hooks

```python
def action_sold(self):
    # Pre-processing
    self._validate_sale()
    
    # Original method
    result = super().action_sold()
    
    # Post-processing
    self._create_invoice()
    
    return result
```

### 2. Conditional Features

```python
def _create_invoice(self):
    if not self.env['ir.module.module'].search([
        ('name', '=', 'account'),
        ('state', '=', 'installed')
    ]):
        return
    
    # Create invoice
```

### 3. Extension Points

```python
def _prepare_invoice_vals(self):
    """Hook method for invoice values"""
    self.ensure_one()
    return {
        'partner_id': self.buyer_id.id,
        'move_type': 'out_invoice',
        'journal_id': self._get_default_journal().id,
    }
```

## Exercises

1. Create Estate Account Module:
   - Set up proper dependencies
   - Create basic structure
   - Test installation/uninstallation

2. Implement Invoice Creation:
   - Override `action_sold`
   - Create basic invoice
   - Add invoice lines

3. Add Advanced Features:
   - Custom invoice lines
   - Validation rules
   - Error handling

## Debugging Tips

1. **Module Installation**:
   - Check module dependencies
   - Verify module loading order
   - Check for missing imports

2. **Invoice Creation**:
   - Use `print` or debugger to check values
   - Verify required fields
   - Check journal configuration

3. **Common Issues**:
   - Missing dependencies
   - Incorrect field values
   - Permission issues

Remember that module interaction requires careful consideration of dependencies and proper error handling to ensure robust functionality.
