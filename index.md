## AdoPiSoft Plugin Developer Guides

**ADOPISOFT** (formerly Ado Piso WiFi) is the leading management software for piso wifi vendo machines world wide. Designed to be easy to use even for non-technical individuals with room for advanced settings and configuration.

Manage your bandwidth, users and rates anytime, anywhere with our remote management tool. Be part of our huge and helpful community and post your questions, suggestions and ideas to improve the system.

### Plugin Files Structure

```
> ROOT_DIR
  > app-runtime
    > release
      > @adopisoft
        > plugins
          > PLUGIN_NAME
            > app
              > models
                  - see MODEL guides
              > controllers
              router.js
              index.js
            > assets
              > scripts
                > admin
                    - admin javascript files here -
                > portal
                    - captive portal javascript files here -
              > fonts
              > images
              > styles
                > admin
                    - admin css files here -
                > portal
                    - captive portal css files here -
            > migrations
                - database migration files here -
            package.json
                - Follow proper package.json formatting, see guides below -
```

### Available App Modules
```
  {
    boot,
    config: {
      version
    },
    machine,
    server_api,
    eloading,
    device_register,
    devices_manager,
    sessions_manager,
    net_checker,
    coinslot_data_decryptor,
    classes: {
      MobileDevice,
      Session
    },
    libs: {
      http,
      plugin_api,
      jwt,
      encryptor
    }
   }
```

Sample usage:
```
  const { machine } = require('@adopisoft/exports')
  
  // get the machine's device ID
  const machine_id = await machine.getId()
  
```


### Available Core Modules
Core source codes are not encrypted and are readily available if you need them.

Core files are located in **ROOT_DIR/release/@adopisoft/core/**

Please see example usage below:
```
  var application = require('@adopisoft/core/config/application.js')
  
  // get the machine's hardware type
  let { hardware } = await application.read()
```

### Plugin Initialization

PLUGIN_ROOT_DIR/index.js
```
module.exports = require('./app')
```

PLUGIN_ROOT_DIR/app/index.js
```
const router = require('./router.js')
const { app } = require('@adopisoft/exports')
module.exports = {
  async init () {
    await models.init()
    app.use(router)
  }
}
```

### Model Guides
**Sample Usage**

app/models/index.js
```
// app/models/index.js

const core_models = require('@adopisoft/core/models')
const { machine } = require('@adopisoft/exports')
const ChargingSession = require('./charging_session.js')

const model_files = {
  ChargingSession
}

exports.init = async () => {
  const {sequelize, models, Sequelize} = await core_models.getInstance()
  const db = await sequelize.getInstance()
  const machine_id = await machine.getId()

  var keys = Object.keys(model_files)
  for (var i = 0; i < keys.length; i++) {
    var k = keys[i]
    models[k] = model_files[k](db, Sequelize)
  }

  var default_scope = {
    where: { machine_id }
  }

  models.ChargingSession.addScope('default_scope', default_scope)
  models.ChargingSession.belongsTo(models.MobileDevice)
  models.MobileDevice.hasMany(models.ChargingSession)

  return { sequelize, models, Sequelize }
}

```

### Database Migration
http://sequelize.org
1. Create migration file using sequelize
> sequelize-cli migration:generate --name create_charging_sessions

2. Edit the generated migration file, see example below:

```
//PLUGIN_ROOT_DIR/migrations/20220125111621-create_charging_sessions.js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('charging_sessions', {
      id: {
        primaryKey: true,
        type: Sequelize.UUID,
        defaultValue: Sequelize.UUIDV4,
        allowNull: false,
        unique: true
      },
      machine_id: {
        type: Sequelize.STRING
      },
      mobile_device_id: {
        type: Sequelize.UUID
      },
      charging_port_id: {
        allowNull: true,
        type: Sequelize.INTEGER
      },
      time_seconds: {
        type: Sequelize.INTEGER,
        defaultValue: 0
      },
      running_time_seconds: {
        type: Sequelize.INTEGER,
        defaultValue: 0
      },
      expire_minutes: {
        allowNull: true,
        type: Sequelize.INTEGER
      },
      expiration_date: {
        allowNull: true,
        type: Sequelize.DATE
      },
      created_at: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updated_at: {
        allowNull: false,
        type: Sequelize.DATE
      }
    })
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('charging_sessions');
  }
}

```


### Proper Formatting of package.json
#### Required attributes
- name
- repository
- metadata (admin->navigation)
  
See example below:
```
{
  "name": "charging-station",
  "version": "1.0.0",
  "description": "Charging station plugin for AdoPiSoft",
  "main": "index.js",
  "repository": "git@github.com:AdoPiSoft-Plugins/charging-station.git",
  "author": "Arnel Lenteria <arnel.lenteria@gmail.com>",
  "license": "MIT",
  "metadata": {
    "admin": {
      "navigation": [
        {
          "parentstate": "plugins",
          "icon": "fa fa-plug",
          "text": "Charging Station",
          "href": "plugins.charging_station"
        }
      ]
    }
  },
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "^4.17.1",
    "ini": "^1.3.5"
  }
}
```


Here's a sample working plugin source codes:
[Charging Station Plugin](https://github.com/AdoPiSoft-Plugins/charging-station/tree/development)

