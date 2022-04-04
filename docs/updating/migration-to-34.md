---
category: updating
menu-title: Migration to v34.x
order: 90
modified_at: 2022-04-12
---

# Migration to CKEditor 5 v34.0.0

<info-box>
	When updating your CKEditor 5 installation, make sure **all the packages are the same version** to avoid errors.

	For custom builds, you may try removing the `package-lock.json` or `yarn.lock` files (if applicable) and reinstalling all packages before rebuilding the editor. For best results, make sure you use the most recent package versions.
</info-box>

For the entire list of changes introduced in version 34.0.0, see the [changelog for CKEditor 5 v34.0.0]

Listed below are the most important changes that require your attention when upgrading to CKEditor 5 v34.0.0.

## Important changes

### Additional dependencies in CKEditor 5 Collaboration Features

The {@link installation/advanced/dll-builds DLL builds} support was introduced for revision history. As a result, some imports, plugin requirements and cross-package dependencies have changed to allow for the new building process.

From now on, additional plugins will be required, when the following CKEditor 5 collaboration features are added to the editor:

* **{@link module:real-time-collaboration/realtimecollaborativerevisionhistory~RealTimeCollaborativeRevisionHistory}** will also require adding {@link module:revision-history/revisionhistory~RevisionHistory} to the list of the editor plugins:

	```js
	// ❌ Old imports:
	import RealTimeCollaborativeRevisionHistory from '@ckeditor/ckeditor5-real-time-collaboration/src/realtimecollaborativerevisionhistory';
	// ✅ New imports:
	import RealTimeCollaborativeRevisionHistory from '@ckeditor/ckeditor5-real-time-collaboration/src/realtimecollaborativerevisionhistory';
	import RevisionHistory from '@ckeditor/ckeditor5-revision-history/src/revisionhistory';
	```

### Changed mechanism for setting and clearing the editor read-only mode

With this update, {@link module:core/editor/editor~Editor#isReadOnly `editor.isReadOnly`} becomes a read-only property. Setting it manually is no longer permitted and will result in an error.

Changing the editor mode it now possible only via dedicated methods that introduce a lock mechanism. Thanks to that, various features that set the read-only mode will not collide and will not have to know about each other. Basically, the editor will become editable again only if all features (that earlier set it to read-only) will allow for that.

The new methods on the `Editor` class are {@link module:core/editor/editor~Editor#enableReadOnlyMode `editor.enableReadOnlyMode( lockId )`}  and {@link module:core/editor/editor~Editor#disableReadOnlyMode `editor.disableReadOnlyMode( lockId )`}, which respectively enable and disable the read-only mode.

The lock mechanism makes the editor read-only if there is at least one lock. If all locks are removed, the content becomes editable again. Because each feature is responsible only for setting and removing its own lock, the features do not conflict each other. Earlier, such issues could result in the editor content being editable when it shouldn't.

	```js
	// ❌ Old usage:
	function makeEditorReadOnly() {
		editor.isReadOnly = true;
	}

	function makeEditorEditable() {
		editor.isReadOnly = false;
	}
	
	// ✅ New usage:
	const myFeatureLockId = Symbol( 'my-feature' );

	function makeEditorReadOnly() {
		editor.enableReadOnlyMode( myFeatureLockId );
	}

	function makeEditorEditable() {
		editor.disableReadOnlyMode( myFeatureLockId );
	}
	```