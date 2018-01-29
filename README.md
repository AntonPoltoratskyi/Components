# Components

[![Swift](https://img.shields.io/badge/Swift-4.0-orange.svg)](https://swift.org)
[![Xcode](https://img.shields.io/badge/Xcode-9.0-blue.svg)](https://developer.apple.com/xcode)
[![MIT](https://img.shields.io/badge/License-MIT-red.svg)](https://opensource.org/licenses/MIT)

This repository will contain some useful iOS code utils and extensions.

## Requirements:
- **iOS** 8.0
- Xcode 9.0+
- Swift 4.0+

## UI Layer

1. **Work with UITableView & UICollectionView** - one possible approach, inspired by **CocoaHeads**:

You can move configuration logic for **UITableViewCell** or **UICollectionViewCell** from **-cellForRowAtIndexPath:** to separate types.

First, you need to create cell class and appropriate **CellViewModel** type.

> UserTableViewCell.swift

```Swift
// MARK: - View Model

struct UserCellModel: CellViewModel {
    var user: User
    
    func setup(cell: UserTableViewCell) {
        cell.nameLabel.text = user.name
    }
}

// MARK: - Cell

final class UserTableViewCell: UITableViewCell {
    @IBOutlet weak var nameLabel: UILabel!
}
```

After that you need to register created model type:

```Swift
tableView.register(nibModel: UserCellModel.self)
```

Then store your models in array (or your custom datasource type):

```Swift
private var users: [AnyCellViewModel] = []
```

**AnyCellViewModel** is a base protocol of **CellViewModel**. 
It's needed only in order to fix compiler limitation as **you can use protocols with associatedtype only as generic constraints** 
and can't write something like this:

```Swift
private var users: [CellViewModel] = [] // won't compile
```

**UITableViewDataSource** implementation is very easy, event if you have multiple cell types, because all logic are contained in our view models:

```Swift
class ViewController: UIViewController {
    
    @IBOutlet weak var tableView: UITableView!
    
    private var users: [AnyCellViewModel] = []

    override func viewDidLoad() {
        super.viewDidLoad()
        users = User.testDataSource.map { UserCellModel(user: $0) }
        tableView.register(nibModel: UserCellModel.self)
    }
}

extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return users.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        return tableView.dequeueReusableCell(withModel: tableModel(at: indexPath), for: indexPath)
    }
    
    private func tableModel(at indexPath: IndexPath) -> AnyCellViewModel {
        return users[indexPath.row]
    }
}
```

See more implementation details in **Sources** folder.
