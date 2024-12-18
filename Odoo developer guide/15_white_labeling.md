# Chapter 15: White-Labeling Odoo

## Important Guidelines

> **CRITICAL: Never modify the existing Odoo codebase directly!**
> 
> 1. Always use inheritance to extend functionality
> 2. All new modules or extended classes MUST start with "onebuilder_"
> 3. Maintain this naming convention for consistency and clarity
> 4. Use proper inheritance mechanisms to ensure updates don't break customizations

For example, instead of:
- `custom_branding` → use `onebuilder_branding`
- `CustomHome` → use `OnebuilderHome`

## Module Structure

```
onebuilder_branding/
├── __init__.py
├── __manifest__.py
├── controllers/
│   ├── __init__.py
│   └── main.py
├── static/
│   ├── src/
│   │   ├── img/
│   │   │   ├── favicon.ico
│   │   │   └── logo.png
│   │   └── scss/
│   │       └── onebuilder_branding.scss
├── views/
│   ├── assets.xml
│   ├── webclient_templates.xml
│   └── login_templates.xml
└── data/
    └── res_config_settings.xml
```

## Manifest Configuration

```python
{
    'name': 'OneBuilder Branding',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'White-label customization for Odoo',
    'sequence': 1,
    'description': """
        This module replaces Odoo branding with OneBuilder branding elements.
        It handles:
        - Login page customization
        - System name changes
        - Logo replacement
        - Favicon updates
        - Footer modifications
    """,
    'depends': ['web', 'base_setup'],
    'data': [
        'views/webclient_templates.xml',
        'views/login_templates.xml',
        'views/assets.xml',
        'data/res_config_settings.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'onebuilder_branding/static/src/scss/onebuilder_branding.scss',
        ],
    },
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}
```

## Template Inheritance

### 1. Web Client Template (webclient_templates.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Override Main Layout -->
    <template id="onebuilder_layout" inherit_id="web.layout">
        <xpath expr="//title" position="replace">
            <title t-esc="title or 'OneBuilder'"/>
        </xpath>
        
        <!-- Replace favicon -->
        <xpath expr="//link[@rel='shortcut icon']" position="replace">
            <link rel="shortcut icon" href="/onebuilder_branding/static/src/img/favicon.ico" type="image/x-icon"/>
        </xpath>
    </template>

    <!-- Override Backend Layout -->
    <template id="onebuilder_menu" inherit_id="web.menu">
        <xpath expr="//div[hasclass('o_main_navbar')]" position="replace">
            <div class="o_main_navbar">
                <a href="#" class="o_menu_brand">OneBuilder</a>
                <t t-call="web.menu_secondary"/>
            </div>
        </xpath>
    </template>

    <!-- Override Footer -->
    <template id="onebuilder_footer" inherit_id="web.footer">
        <xpath expr="//div[@class='o_footer']" position="replace">
            <div class="o_footer">
                <span>OneBuilder &copy; 2024</span>
            </div>
        </xpath>
    </template>
</odoo>
```

### 2. Login Page Template (login_templates.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="onebuilder_login_layout" inherit_id="web.login_layout">
        <xpath expr="//div[@class='container']" position="replace">
            <div class="container">
                <div class="card">
                    <div class="card-body">
                        <div class="text-center">
                            <img t-attf-src="/onebuilder_branding/static/src/img/logo.png" 
                                 alt="OneBuilder Logo" 
                                 style="max-width: 150px;"/>
                        </div>
                        <t t-raw="0"/>
                    </div>
                </div>
            </div>
        </xpath>
    </template>
</odoo>
```

### 3. Assets Configuration (assets.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="onebuilder_assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/onebuilder_branding/static/src/scss/onebuilder_branding.scss"/>
        </xpath>
    </template>
</odoo>
```

## Python Controllers

### Main Controller (controllers/main.py)
```python
from odoo import http
from odoo.addons.web.controllers.main import Home

class OnebuilderHome(Home):
    @http.route()
    def index(self, *args, **kw):
        # Inherit and extend homepage functionality
        response = super().index(*args, **kw)
        return response

    @http.route()
    def web_client(self, s_action=None, **kw):
        # Inherit and extend web client functionality
        response = super().web_client(s_action=s_action, **kw)
        return response
```

## System Parameters (data/res_config_settings.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <!-- System Name -->
        <record id="system_name" model="ir.config_parameter">
            <field name="key">web.base.title</field>
            <field name="value">OneBuilder</field>
        </record>
        
        <!-- System Publisher -->
        <record id="system_publisher" model="ir.config_parameter">
            <field name="key">web.base.publisher</field>
            <field name="value">OneBuilder</field>
        </record>
    </data>
</odoo>
```

## SCSS Customization (static/src/scss/onebuilder_branding.scss)
```scss
// Override Odoo branding colors and styles
.o_main_navbar {
    background-color: #your-color !important;
}

// Hide Odoo branding elements
.o_odoo_logo, 
.o_brand_promotion {
    display: none !important;
}

// Custom styling for login page
.o_login_page {
    background-color: #your-background-color;
}
```

## Installation and Update Process

1. Place the module in your addons path
2. Update the module list in Odoo
3. Install the module
4. Restart the Odoo server
5. Clear browser cache

## Maintenance Notes

1. **Version Compatibility**
   - Test after Odoo updates
   - Review template changes in new versions
   - Update XPath expressions if needed

2. **Asset Management**
   - Keep brand assets in module
   - Use appropriate image formats
   - Optimize file sizes

3. **Security Considerations**
   - Maintain proper access rights
   - Protect brand assets
   - Follow Odoo security guidelines

4. **Update Strategy**
   - Document all customizations
   - Keep original templates as reference
   - Test thoroughly before deployment

Remember that this white-labeling approach uses proper inheritance, ensuring your customizations persist through Odoo updates while maintaining the ability to upgrade the core system.
