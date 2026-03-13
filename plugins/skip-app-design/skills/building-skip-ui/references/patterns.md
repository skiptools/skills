# Common UI Patterns

## Tab-Based Navigation

```swift
struct AppTabView: View {
    var body: some View {
        TabView {
            HomeView()
                .tabItem { Label("Home", systemImage: "house") }
            SearchView()
                .tabItem { Label("Search", systemImage: "magnifyingglass") }
            ProfileView()
                .tabItem { Label("Profile", systemImage: "person") }
        }
    }
}
```

## List with Navigation Detail

```swift
struct ItemListView: View {
    let items: [Item]

    var body: some View {
        NavigationStack {
            List(items) { item in
                NavigationLink(value: item) {
                    HStack {
                        Image(systemName: item.icon)
                        VStack(alignment: .leading) {
                            Text(item.name).font(.headline)
                            Text(item.subtitle).font(.subheadline).foregroundStyle(.secondary)
                        }
                    }
                }
            }
            .navigationDestination(for: Item.self) { item in
                ItemDetailView(item: item)
            }
            .navigationTitle("Items")
        }
    }
}
```

## Async Data Loading with Pull-to-Refresh

```swift
struct DataView: View {
    @State private var items: [Item] = []
    @State private var isLoading = true
    @State private var errorMessage: String?

    var body: some View {
        Group {
            if isLoading {
                ProgressView()
            } else if let error = errorMessage {
                VStack(spacing: 12) {
                    Text(error).foregroundStyle(.secondary)
                    Button("Retry") { Task { await load() } }
                }
            } else {
                List(items) { item in
                    Text(item.name)
                }
                .refreshable { await load() }
            }
        }
        .task { await load() }
    }

    private func load() async {
        isLoading = items.isEmpty
        do {
            items = try await fetchItems()
            errorMessage = nil
        } catch {
            errorMessage = error.localizedDescription
        }
        isLoading = false
    }
}
```

## Form with Validation

```swift
struct SettingsView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var notifications = true
    @State private var frequency = "daily"

    var body: some View {
        NavigationStack {
            Form {
                Section("Profile") {
                    TextField("Name", text: $name)
                    TextField("Email", text: $email)
                        .textInputAutocapitalization(.never)
                }
                Section("Preferences") {
                    Toggle("Enable Notifications", isOn: $notifications)
                    if notifications {
                        Picker("Frequency", selection: $frequency) {
                            Text("Daily").tag("daily")
                            Text("Weekly").tag("weekly")
                            Text("Monthly").tag("monthly")
                        }
                    }
                }
                Section {
                    Button("Save") { save() }
                        .disabled(name.isEmpty || email.isEmpty)
                }
            }
            .navigationTitle("Settings")
        }
    }
}
```

## Search with Filtering

```swift
struct SearchableListView: View {
    @State private var searchText = ""
    let items: [Item]

    var filteredItems: [Item] {
        if searchText.isEmpty { return items }
        return items.filter { $0.name.localizedCaseInsensitiveContains(searchText) }
    }

    var body: some View {
        NavigationStack {
            List(filteredItems) { item in
                Text(item.name)
            }
            .searchable(text: $searchText, prompt: "Search items")
            .navigationTitle("Items")
        }
    }
}
```

## Sheet Presentation

```swift
struct ParentView: View {
    @State private var showDetail = false
    @State private var selectedItem: Item?

    var body: some View {
        Button("Show Detail") { showDetail = true }
            .sheet(isPresented: $showDetail) {
                DetailSheet()
            }
            .sheet(item: $selectedItem) { item in
                ItemDetailSheet(item: item)
            }
    }
}
```

## Observable Model with Async

```swift
import Observation

@Observable final class TodoViewModel {
    var todos: [Todo] = []
    var isLoading = false
    var errorMessage: String?

    func load() async {
        isLoading = true
        defer { isLoading = false }
        do {
            todos = try await TodoService.fetchAll()
        } catch {
            errorMessage = error.localizedDescription
        }
    }

    func add(_ title: String) async {
        let todo = Todo(title: title)
        todos.append(todo)
        try? await TodoService.create(todo)
    }

    func delete(_ todo: Todo) async {
        todos.removeAll { $0.id == todo.id }
        try? await TodoService.delete(todo)
    }
}
```

```swift
struct TodoListView: View {
    @State private var viewModel = TodoViewModel()
    @State private var newTitle = ""

    var body: some View {
        NavigationStack {
            List {
                ForEach(viewModel.todos) { todo in
                    Text(todo.title)
                }
                .onDelete { indexSet in
                    for index in indexSet {
                        Task { await viewModel.delete(viewModel.todos[index]) }
                    }
                }
            }
            .overlay {
                if viewModel.isLoading { ProgressView() }
            }
            .toolbar {
                ToolbarItem(placement: .bottomBar) {
                    HStack {
                        TextField("New todo", text: $newTitle)
                        Button("Add") {
                            Task { await viewModel.add(newTitle) }
                            newTitle = ""
                        }
                        .disabled(newTitle.isEmpty)
                    }
                }
            }
            .task { await viewModel.load() }
            .navigationTitle("Todos")
        }
    }
}
```
