# MovableTableViewCell
In this tutorial you will learn how to make a moveable table view cell by using a long press gesture.

## How to do it?
* [Add `UILongGestureRecognizer`](#add-longGestureRecognizer)
* [Handle `UIGestureRecognizerState.began`](#handle-began)
* [Handle `UIGestureRecognizerState.changed`](#handle-changed)
* [Clean up](#clean-up)

### Step 1: Add `UILongGestureRecognizer` on `UITableView` / `UICollectionView`: <a name="add-longGestureRecognizer"></a>
```swift
override func viewDidLoad() {
  super.viewDidLoad()
  let longPress = UILongPressGestureRecognizer(target: self, action: #selector(ViewController.longPressGestureRecognized(longPress:)))
  self.tableView.addGestureRecognizer(longPress)
}
```
& 
```swift
func longPressGestureRecognized(longPress: UILongPressGestureRecognizer) {
}
```

### Step 2: Handle `UIGestureRecognizerState.began`  <a name="handle-began"></a>
Before this we will need two class variables
```swift
fileprivate var sourceIndexPath: IndexPath?
fileprivate var snapshot: UIView?
```
`sourceIndexPath` to save index path of tableview cell, where gesture begins & `snapshot` to save snapshot of the cell user is moving.
####Handle `.began`
```swift
func longPressGestureRecognized(longPress: UILongPressGestureRecognizer) {
  let state = longPress.state
  let location = longPress.location(in: self.tableView)
  guard let indexPath = self.tableView.indexPathForRow(at: location) else { return }
  switch state {
    case .began:
    sourceIndexPath = indexPath
    guard let cell = self.tableView.cellForRow(at: indexPath) else { return }
    // Take a snapshot of the selected row using helper method. See below method
    snapshot = self.customSnapshotFromView(inputView: cell)
    guard  let snapshot = self.snapshot else { return }
    var center = cell.center
    snapshot.center = center
    snapshot.alpha = 0.0
    self.tableView.addSubview(snapshot)
    UIView.animate(withDuration: 0.25, animations: {
      center.y = location.y
      snapshot.center = center
      snapshot.transform = CGAffineTransform(scaleX: 1.05, y: 1.05)
      snapshot.alpha = 0.98
      cell.alpha = 0.0
    }, completion: { (finished) in
      cell.isHidden = true
    })
    break
    default:
    break
  }
}
```
####Helper method to create snapshot of selected cell.
```swift
private func customSnapshotFromView(inputView: UIView) -> UIView? {
  UIGraphicsBeginImageContextWithOptions(inputView.bounds.size, false, 0)
  if let CurrentContext = UIGraphicsGetCurrentContext() {
    inputView.layer.render(in: CurrentContext)
  }
  guard let image = UIGraphicsGetImageFromCurrentImageContext() else {
    UIGraphicsEndImageContext()
    return nil
  }
  UIGraphicsEndImageContext()
  let snapshot = UIImageView(image: image)
  snapshot.layer.masksToBounds = false
  snapshot.layer.cornerRadius = 0
  snapshot.layer.shadowOffset = CGSize(width: -5, height: 0)
  snapshot.layer.shadowRadius = 5
  snapshot.layer.shadowOpacity = 0.4
  return snapshot
}
```
Using `.began1` case. If there is a valid index path, get the corresponding `UITableViewCell` and take a snapshot view of the table view cell using a helper method. 
Then add the new snapshot view to the table view and center it on the corresponding cell.

### Step 3: Handle `UIGestureRecognizerState.changed`  <a name="handle-changed"></a>
As gesture moves (the `.changed` case), move the snapshot view by offsetting its Y coordinate only. 
If the gesture moves enough that its location corresponds to a different index path, tell the table view to move the rows.
At the same time, you should update your data source too.

```swift
case .changed:
  guard  let snapshot = self.snapshot else {
    return
  }
  var center = snapshot.center
  center.y = location.y
  snapshot.center = center
  guard let sourceIndexPath = self.sourceIndexPath  else {
    return
  }
  if indexPath != sourceIndexPath {
    swap(&data[indexPath.row], &data[sourceIndexPath.row])
    self.tableView.moveRow(at: sourceIndexPath, to: indexPath)
    self.sourceIndexPath = indexPath
  }
break
```

### Step 4: Clean up  <a name="clean-up"></a>

When the gesture either ends or cancels, both the table view and data source are up to date.
All you have to do is removing the snapshot view from table view and undo cell fade out.

```swift
default:
  guard let cell = self.tableView.cellForRow(at: indexPath) else {
    return
  }
  guard  let snapshot = self.snapshot else {
    return
  }
  cell.isHidden = false
  cell.alpha = 0.0
  UIView.animate(withDuration: 0.25, animations: {
    snapshot.center = cell.center
    snapshot.transform = CGAffineTransform.identity
    snapshot.alpha = 0
    cell.alpha = 1
  }, completion: { (finished) in
    self.cleanup()
})
```
####Create `cleanup()` method
```swift
private func cleanup() {
  self.sourceIndexPath = nil
  snapshot?.removeFromSuperview()
  self.snapshot = nil
}
```
Also clean up(`cleanup()`), if indexPath is invalid.
```swift
func longPressGestureRecognized(longPress: UILongPressGestureRecognizer) {
  let state = longPress.state
  let location = longPress.location(in: self.tableView)
  guard let indexPath = self.tableView.indexPathForRow(at: location) else {
  //Clean up
    self.cleanup()
    return
  }
  switch state {
  //...
  .
  .
  .
  ....//
```

## Demo
You can download the demo.
