# Cross plugin links and commands

Using the Plugin extensions API App, plugins can register placements of their own to display other plugins links. This cross-plugin linking creates a more immersive user experience for installed plugins.

## How to create a placement within a plugin

The `getPluginExtensions` method in `@grafana/runtime` creates a placement within your plugin. A placement is a way of specifying where in the plugin UI other plugins links or commands are rendered. `getPluginExtensions` takes an object consisting of the `placement` name, which must begin `plugin/<pluginId>`, and include any contextual information you would like to provide for the use of other plugins. It returns a list of `extensionLinks` and `extensionCommands` that your program can loop over.

_Note: Creating a placement in a plugin creates a public interface for other plugins to interact with. Changes to the placement or the context it passes could break any plugin that attempts to register a link inside your plugin._

```typescript
import { getPluginExtensions } from '@grafana/runtime';
import { isPluginExtensionLink, isPluginExtensionCommand } from '@grafana/data';
import { LinkButton } from '@grafana/ui';

function AppPlacementExample() {
  const { extensions } = getPluginExtensions({
    placement: 'plugin/another-app-plugin/menu', // Must start with "plugin/"
    context: Object.freeze({
      pluginId: 'another-app-plugin',
    }),
  });

  if (extensions.length === 0) {
    return null;
  }

  return (
    <div>
      {extensions.map((extension) => {
        if (isPluginExtensionLink(extension)) {
          return (
            <LinkButton href={extension.path} title={extension.description} key={extension.key}>
              {extension.title}
            </LinkButton>
          );
        }

        if (isPluginExtensionCommand(extension)) {
          return (
            <Button onClick={extension.callHandlerWithContext} title={extension.description} key={extension.key}>
              {extension.title}
            </Button>
          );
        }
        return null;
      })}
    </div>
  );
}
```

In the above example, we created a component that renders `<LinkButton />` or `<Button />` components depending on the type of extension other plugins register for the `plugin/another-app-plugin/menu` placement. We pass the context as the second parameter to `getPluginExtensions`, being careful to make it immutable using the `Object.freeze` method.

## How to add links or commands from another plugin

Create links from other plugins in the same way you would [extend the Grafana application UI]({{< relref "./extend-the-grafana-ui-with-links-and-commands" >}}) with a link or command. Rather than specify a `grafana/...` placement, specify the plugin placement `plugin/<pluginId>/<placementId>` instead. Given the placement example above, use a plugin link and command such as the following.

### Link example

```typescript
new AppPlugin().configureExtensionLink({
  title: 'Go to basic app',
  description: 'Will navigate the user to the basic app',
  placement: 'plugin/another-app-plugin/menu',
  path: '/a/myorg-basic-app/one',
});
```

### Command example

```typescript
new AppPlugin().configureExtensionCommand({
  title: 'Title of the command"',
  description: 'Some basic description...',
  placement: 'plugin/another-app-plugin/menu',
  handler: (context: PanelContext, helpers: any): void => {
    alert('Hello world');
  },
});
```