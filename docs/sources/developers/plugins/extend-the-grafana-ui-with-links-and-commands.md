# Extend the Grafana UI with links and commands

Use the Plugin extensions API with your Grafana App plugins to add links to the Grafana UI. Doing so allows you to direct users to your plugins pages from various "placements" within the Grafana application.

For a plugin to successfully register links it must:

- Be an App plugin.
- Be preloaded.
- Be installed and enabled.

## Add a link extension within Grafana

Here's how you can add a link to the dashboard panel menus in Grafana via your plugin:

Define the link extension in your plugin's `module.ts` file. First, define a new instance of the `AppPlugin` class by using the `configureExtensionLink` method. This method takes an object that describes your link extension, including a `title` property for the link text, a `placement` that tells Grafana where the link should appear, and a `path` for the user to navigate to your plugin.

```typescript
new AppPlugin().configureExtensionLink({
  title: 'Go to basic app',
  description: 'Will navigate the user to the basic app',
  placement: 'grafana/dashboard/panel/menu',
  path: '/a/myorg-basic-app/one', // Must start with "/a/<PLUGIN_ID>/"
});
```

That's it! Your link will be displayed in dashboard panel menus. When the user clicks the link, they will be navigated to the path you defined earlier.

_Note: Each plugin is limited to a maximum of two links per placement._

## Add a link extension using context within Grafana

The above example works for simple cases however you may want to act on information from the panel the user is navigating from. This can be achieved by making use of the `configure` property which takes a function and returns an object that consists of a `title` property for the link text, a `path` to navigate the user to your plugin. Alternatively this function may return `undefined` if there's a need to hide the link for certain scenarios.

```typescript
new AppPlugin().configureExtensionLink({
  title: 'Go to basic app',
  description: 'Will send the user to the basic app',
  placement: 'grafana/dashboard/panel/menu',
  path: '/a/myorg-basic-app/one',
  configure: (link: AppPluginLinkExtension, context: PanelContext) => {
    switch (context?.pluginId) {
      case 'timeseries':
        return {
          title: 'Go to page one',
          description: 'hello',
          path: '/a/myorg-basic-app/one',
        };

      case 'piechart':
        return {
          title: 'Go to page two',
          path: '/a/myorg-basic-app/two',
        };

      // Returning undefined tells Grafana to hide the link
      default:
        return undefined;
    }
  },
});
```

The above example demonstrates how to return a different link `path` based on which plugin the dashboard panel is using. If the clicked-upon panel is neither a timeseries nor a piechart panel, then the configure function returns `undefined` and the link isn't rendered.

_Note: The context passed to the `configure` function is bound by the `placement` the link is inserted into. Different placements contain different contexts._

## Add a command extension within Grafana

Link extensions provide the means to direct users to a plugin page via href links within the Grafana UI. Commands, on the other hand, perform dynamic actions when clicked.

Here's how you can add a command link to the dashboard panel menus in Grafana via your plugin:

1. Define the command extension in the plugin's `module.ts` file.
1. Create a new instance of the `AppPlugin` class, and this time use the `configureExtensionCommand` method. This method takes a `context` object that contains information about the panel where the menu was clicked, and a `helpers` object to help perform various actions.

In the following example, we open a modal.

```typescript
new AppPlugin().configureExtensionCommand({
  title: 'Title of the command"',
  description: 'Some basic description...',
  placement: 'grafana/dashboard/panel/menu',
  handler: (context: PanelContext, helpers: any): void => {
    helpers.openModal({
      title: 'My plugin modal',
      body: ({ onDismiss }) => <SampleModal onDismiss={onDismiss} pluginId={context?.pluginId} />,
    });
  },
});

type Props = {
  onDismiss: () => void;
  pluginId?: string;
};

const SampleModal = ({ onDismiss, pluginId }: Props) => {
  return (
    <VerticalGroup spacing="sm">
      <p>This modal was opened via the plugin extensions API.</p>
      <p>The panel is using a {pluginId} plugin to display data.</p>
    </VerticalGroup>
  );
};
```

The plugin extensions API is a powerful feature for you to insert links into the UI of Grafana applications that send users to plugin features or trigger actions based on where the user clicked. This feature can also be used for [cross plugin linking](./cross-plugin-linking.md).