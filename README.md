## Gaze

A tiny C++ pub/sub library, a single header file. The code is a cyclic mess but it seems to work
well enough. This is suitable for simple projects with a "watchable" data source that, when changed,
should reflect some sort of UI, for example.

The code is relatively thread-safe (I think) but it wasn't tested in a highly concurrent environment,
so who knows.

## Description

There are three classes, `subject`, `watcher` and `source`. Both `watcher` and `source` are
subclasses of `subject`. A `watcher` can subscribe to updates from a `subject` (be that a `source`
or another `watcher`).

## Examples

```cpp
class ViewModel: public gaze::watcher {
public:
    // The constructor that takes `this` as an argument starts watching
    // the source immediately.
    gaze::source<std::string> name{"", this};
    gaze::source<int> age{0, this};

private:
    // Will be called when either `name` or `age` are updated via the `set()` method, since
    // we're watching both.
    virtual void subject_updated(const gaze::subject* subj) override {
        if (subj == this->name) {
            std::printf("name just updated to %s", this->name.get());
        } else if (subj == this->age) {
            std::printf("age just updated to %d", this->age.get());
        }
    }
};

class SomeUI: public gaze::watcher {
public:
    SomeUI(ViewModel* viewModel) {
        this->viewModel = viewModel;
        this->watch(viewModel);
    }

private:
    // Will be called when the `viewModel` is updated, either by `name` or `age`.
    // `subj` in this case will be the entire `viewModel`.
    virtual void subject_updated(const gaze::subject* subj) override {
        // ViewModel just updated, refresh UI or whatever...
    }

private:
    ViewModel* viewModel;
};

class SomeReactiveText: public SomeTextComponent, public gaze::watcher {
public:
    SomeReactiveText(gaze::source<std::string>* value) {
        this->value = this->watch_assign(value);
        // 'watch_assign' is a helper to avoid needing to do this:
        // this->value = value;
        // this->watch(value);
    }

    virtual void subject_updated(const gaze::subject* subj) override {
        this->text = this->value.get(); // Update 'text' from SomeTextComponent...
    }

private:
    const gaze::source<std::string>* value;
};
```

## Notes

This was made to help me with developing wxWidgets applications. wxWidgets is a cool framework
but it's 100% imperative, so everytime you want something to change you need to call
`myView->UpdateSomeValue(newValue)` directly in the UI. Using `gaze` I can just update the data
source and let the helper Views (like `SomeReactiveText`) update themselves since they're watching
the data.

## License

Whatever I don't care
