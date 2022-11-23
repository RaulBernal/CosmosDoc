# COSMOS SDK v0.46 - Add a module (NFT)

## What is the NFT module? 
By default, **Cosmos SDK v0.46** supports several new [modules](https://docs.cosmos.network/main/modules). Some of them, such as the NFT module, must be manually added to your source code. An update must be performed to include it in the `Core` and `Storage`.

![](https://i.imgur.com/mYi4p7B.png)


In this document we will explain also how to add the storage for the `NFT` module. Specifically, it shows how to upgrade from a v0.45.x based chain to v0.46 but you could also do it over an existent chain v0.46.

>Note: When you find a piece of code like this, it means that you should add the code to the existent code, in the same way you found:
```golang
    ...
    nftmodule.AppModuleBasic{},  // add to the existent list 
    ...   
```
![](https://i.imgur.com/aNFCxh1.png)

### Sources: 
- https://docs.cosmos.network/main/modules/nft
- https://github.com/BitCannaGlobal/bcna/blob/SDK_046/app/app.go


## What is the `x/nft` SDK v0.46 module? 
`x/nft` is an implementation of a Cosmos SDK module, per [ADR 43](https://github.com/cosmos/cosmos-sdk/blob/main/docs/architecture/adr-043-nft-module.md), that allows you to create nft classification, create NFT, transfer NFT, update NFT, and support various queries by integrating the module. It is fully compatible with the ERC721 specification.

## How to proceed to include the NFT module in a Cosmos chain based in SDK v0.46?
Everything that needs to be coded is in app/app.go when the rest of the code is ready for the [SDK v0.46](https://github.com/cosmos/cosmos-sdk/blob/main/UPGRADING.md#v046x).
So let's open the `app/app.go` file and code the following!

### 1) Import keeper and module from the SDK repo:
```golang
    minttypes "github.com/cosmos/cosmos-sdk/x/mint/types"
    ...
    "github.com/cosmos/cosmos-sdk/x/nft"
    nftkeeper "github.com/cosmos/cosmos-sdk/x/nft/keeper"
    nftmodule "github.com/cosmos/cosmos-sdk/x/nft/module"
    ...
    "github.com/cosmos/cosmos-sdk/x/params"
```

### 2) Add the module to `ModuleBasics = module.NewBasicManager`
```golang
    ...
    nftmodule.AppModuleBasic{},
    ...   
```

### 3) Add the module account permissions: `maccPerms`
```golang
    ...
    nft.ModuleName: nil,
    ...
```

### 4)Add keeper as type in `App struct`
```golang
    ...
    NFTKeeper        nftkeeper.Keeper
    ...
```

### 5) Add to Storage keys: `keys := sdk.NewKVStoreKeys`

```golang
    ...
    nftkeeper.StoreKey,
    ...
```

### 6) Now add to keepers functions this new one:
Could be placed after the `app.UpgradeKeeper = upgradekeeper.NewKeeper()` in example.
```golang
    ...
    app.NFTKeeper = nftkeeper.NewKeeper(
        keys[nftkeeper.StoreKey], 
        appCodec, 
        app.AccountKeeper, 
        app.BankKeeper,
    )
    ...
```

### 7) Add to `app.mm = module.NewManager` 
```golang
    ...
    nftmodule.NewAppModule(appCodec, app.NFTKeeper, app.AccountKeeper, app.BankKeeper, app.interfaceRegistry),
    ...
```

### 8) Finally add 3 times in `SetOrderBeginBlockers`, `SetOrderEndBlockers` & `SetOrderInitGenesis`

```golang
    ...
    nft.ModuleName,
    ...
```

## UPGRADE Handler
Let's write an upgrade handler function that you could call in your code to perform the modules upgrades and module additions. Specifically, you need to set also a new type in the 'Store' (line 14)

```golang=
func (app *App) RegisterUpgradeHandlers() {
	planName := "wakeandbake"
	app.UpgradeKeeper.SetUpgradeHandler(planName, func(ctx sdk.Context, plan upgradetypes.Plan, fromVM module.VersionMap) (module.VersionMap, error) {
		ctx.Logger().Info("start to run module migrations...")

		return app.mm.RunMigrations(ctx, app.configurator, fromVM)
	})

	upgradeInfo, err := app.UpgradeKeeper.ReadUpgradeInfoFromDisk()
	if err != nil {
		panic(err)
	}

	if upgradeInfo.Name == planName && !app.UpgradeKeeper.IsSkipHeight(upgradeInfo.Height) {
		storeUpgrades := storetypes.StoreUpgrades{
			Added: []string{
				group.ModuleName, // you can add it if the module is included
				nft.ModuleName,
 			},
		}

		// Configure store loader that checks if version == upgradeHeight and applies store upgrades
		app.SetStoreLoader(upgradetypes.UpgradeStoreLoader(upgradeInfo.Height, &storeUpgrades))
	}
}
```
### And that's it!
![](https://i.imgur.com/R4j9Jwp.png)

and...

![](https://i.imgur.com/XkC1jE0.png)
