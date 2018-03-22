# TDXDemo
Basic Mobile SDK Swift template modified to be offline first.
The application was generated with:
```shell
forceios create --apptype=native_swift --appname=TDXDemo --packagename=com.acme --organization=Acme --outputdir=
```
**You can see the application at that stage by checking out tag `1_START`.**

## To get started do the following from the root directory
``` shell
node ./install.js
```
## To run the application
* Open `TDXDemo.xcworkspace` in XCode

## Demo steps

### Add schema and sync config
Add `userstore.json` file to TDXDemo target
```json
{
  "soups": [
    {
      "soupName": "User",
      "indexes": [
        { "path": "Id", "type": "string"},
        { "path": "Name", "type": "string"},
        { "path": "__local__", "type": "string"}
      ]
    }
  ]
}
```
Add `usersyncs.json` to TDXDemo target
```json
{
  "syncs": [
    {
      "syncName": "syncDownUsers",
      "syncType": "syncDown",
      "soupName": "User",
      "target": {"type":"soql", "query":"SELECT Name FROM User LIMIT 100"},
      "options": {"mergeMode":"OVERWRITE"}
    }
  ]
}
```
Add following lines in `AppDelegate.m` in `.postLaunch` block 
```swift
SalesforceSwiftSDKManager.shared().setupUserStoreFromDefaultConfig()
SalesforceSwiftSDKManager.shared().setupUserSyncsFromDefaultConfig()
```

**You can see the application at that stage by checking out tag `2_CONFIGS`.**

### Run sync at startup 
Replace loadView with following lines in `RootViewController.swift`
```swift
    var store: SFSmartStore?
    var syncManager: SFSmartSyncSyncManager?
    
    // MARK: - View lifecycle
    override func loadView()
    {
        super.loadView()
        self.title = "Mobile SDK Sample App"
        
        store = SFSmartStore.sharedStore(withName: kDefaultSmartStoreName) as?  SFSmartStore
        syncManager = SFSmartSyncSyncManager.sharedInstance(for:store!)

        // Run (delta)sync if possible
        _ = syncManager?.Promises
            .reSync(syncName: "syncDownUsers")
    }
```

**You can see the application at that stage by checking out tag `3_SYNC`.**

### Load data from store

In loadView, call `self.loadFromStore()`  at startup and when sync completes.

```swift
        // Run (delta)sync if possible
        _ = syncManager?.Promises
            .reSync(syncName: "syncDownUsers")
            .done { [unowned self] (_) in
                self.loadFromStore();
            }
        self.loadFromStore();
```

Add the loadFromStore method.

```swift
    // MARK: - Loading from smartstore
    func loadFromStore()
    {
        let querySpec = SFQuerySpec.Builder(soupName:"User")
            .queryType(value: "smart")
            .smartSql(value: "select {User:Name} from {User}")
            .pageSize(value: 100)
            .build();
        
        _ = self.store?.Promises
            .query(querySpec: querySpec, pageIndex: 0)
            .done { records in
                self.dataRows = (records as! [[NSString]]).map({ row in
                    return ["Name": row[0]];
                })
                DispatchQueue.main.async(execute: {
                    self.tableView.reloadData()
                })
            }
            .catch { error in
                SalesforceSwiftLogger.log(type(of:self), level:.debug, message:"Error: \(error)")
            }
    }
```

**You can see the application at that stage by checking out tag `4_OFFLINE`.**
