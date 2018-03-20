# TDXDemo
Basic Mobile SDK Swift template modified to be offline first.
The application was generated with:
`forceios create --apptype=native_swift --appname=TDXDemo --packagename=com.acme --organization=Acme --outputdir=`

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

### Run sync at startup and load data from store
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

        self.loadFromStore()
        
        // Run (delta)sync if possible
        _ = syncManager?.Promises
            .reSync(syncName: "syncDownUsers")
            .done { [unowned self] (_) in
                self.loadFromStore()
            }
    }
    
    // MARK: - Loading from smartstore
    func loadFromStore()
    {
        let querySpec = SFQuerySpec.Builder(soupName:"User")
            .queryType(value:"range")
            .orderPath(value:"Name")
            .order(value:"ascending")
            .build();
        
        _ = self.store?.Promises
            .query(querySpec: querySpec, pageIndex: 0)
            .done { records in
                self.dataRows = records as! [NSDictionary];
                DispatchQueue.main.async(execute: {
                    self.tableView.reloadData()
                })
            }
            .catch { error in
                SalesforceSwiftLogger.log(type(of:self), level:.debug, message:"Error: \(error)")
            }
    }
```
