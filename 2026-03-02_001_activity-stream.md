
**Ref:** main
**Depends:** 2026-03-01_001_more-ui-updates

### Context

Clicking the activity/pulse icon reveals an activity stream panel like so
![[Pasted image 20260302002648.png|300]]

**Sample panel item html**

```html
<li tabindex="-1" role="listitem" ng-if="::!isFormStream" class="h-card h-card_md h-card_comments ng-scope" ng-click="controls.showRecord($event, entry, sysId)" ng-animate="'sn-animate-stream-entry'" ng-repeat="entry in entries" ng-include="entryTemplate">
	
	<div ng-init="user_message = true" role="group" aria-label="Stream Entry System Administrator 2025-12-21 02:30:57" class="ng-scope">
		<div class="sn-card-component sn-card-component_first sn-card-component_headline ng-binding">
			KB0000051 v1.0
			<!-- ngIf: !controls.getTitle(entry) --></div>

		<!-- ngIf: entry.attachment && entry.attachment.state == 'not_available' -->

		<!-- ngIf: entry.attachment && entry.type == 'attachment-image' &&       entry.attachment.thumbnail_path && entry.attachment.state != 'not_available' -->

		<!-- ngRepeat: journal in ::entry.entries.journal --><!-- ngIf: entry.attachment && entry.type != 'attachment-image' && entry.attachment.state != 'not_available' --><!-- ngIf: ::entry.entries.changes.length > 0 --><div class="sn-card-component sn-card-component_records ng-scope" ng-if="::entry.entries.changes.length &gt; 0"><div class="sn-widget"><ul class="sn-widget-list sn-widget-list-table"><!-- ngRepeat: change in ::entry.entries.changes --><li ng-repeat="change in ::entry.entries.changes" class="ng-scope"><span class="sn-widget-list-table-cell" sn-bind-once="change.field_label">Knowledge base</span><span class="sn-widget-list-table-cell"><!-- ngIf: change.old_value && auditMessageParts.prefix --><!-- ngIf: change.new_value && isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><span ng-if="change.new_value &amp;&amp; !isHTMLField(change) &amp;&amp; (auditMessageParts.newValPosition == '0' || !change.old_value)" class="sn-stream-bound-value ng-binding ng-scope" ng-bind="::change.new_value">Known Error</span><!-- end ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.middle --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.postfix --></span></li><!-- end ngRepeat: change in ::entry.entries.changes --><li ng-repeat="change in ::entry.entries.changes" class="ng-scope"><span class="sn-widget-list-table-cell" sn-bind-once="change.field_label">Short description</span><span class="sn-widget-list-table-cell"><!-- ngIf: change.old_value && auditMessageParts.prefix --><!-- ngIf: change.new_value && isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><span ng-if="change.new_value &amp;&amp; !isHTMLField(change) &amp;&amp; (auditMessageParts.newValPosition == '0' || !change.old_value)" class="sn-stream-bound-value ng-binding ng-scope" ng-bind="::change.new_value">USB port is not working on my PC</span><!-- end ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.middle --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.postfix --></span></li><!-- end ngRepeat: change in ::entry.entries.changes --><li ng-repeat="change in ::entry.entries.changes" class="ng-scope"><span class="sn-widget-list-table-cell" sn-bind-once="change.field_label">Workflow</span><span class="sn-widget-list-table-cell"><!-- ngIf: change.old_value && auditMessageParts.prefix --><!-- ngIf: change.new_value && isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><span ng-if="change.new_value &amp;&amp; !isHTMLField(change) &amp;&amp; (auditMessageParts.newValPosition == '0' || !change.old_value)" class="sn-stream-bound-value ng-binding ng-scope" ng-bind="::change.new_value">Published</span><!-- end ngIf: change.new_value && !isHTMLField(change) && (auditMessageParts.newValPosition == '0' || !change.old_value) --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.newValPosition == '1' --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.middle --><!-- ngIf: ::!change.new_value && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '0' --><!-- ngIf: change.old_value && isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && !isHTMLField(change) && auditMessageParts.oldValPosition == '1' --><!-- ngIf: change.old_value && auditMessageParts.postfix --></span></li><!-- end ngRepeat: change in ::entry.entries.changes --></ul></div></div><!-- end ngIf: ::entry.entries.changes.length > 0 --><!-- ngIf: ::entry.entries.custom.length > 0 -->

		<div>
			<button class="list2-edit-button btn btn-default" ng-click="controls.showRecord($event, entry, sysId)" tabindex="0">
				Open Entry
				<span class="sr-only ng-binding">KB0000051 v1.0</span></button></div>

		<div class="sn-card-component sn-card-component_meta sn-card-component_meta_sibling">
			<span class="sn-card-component-avatar avatar-xs avatar-container ng-isolate-scope" ng-class="avatarType()" primary-non-assign="{avatar: entry.user_image, initials: entry.initials, userID: entry.user_id, displayName: entry.sys_created_by}" show-presence="true"><!-- ngIf: !noPopover() --><a class="sn-popover-complex ng-scope" role="presentation" ng-style="{'cursor': popoverCursor}" style="background-color: transparent; cursor: pointer;" ng-if="!noPopover()" template="sn_avatar_content.xml" button-template="sn_avatar_button.xml"><!-- ngIf: users.length > 1 && !groupAvatar --><div class="avatar soloAvatar bottom"><div aria-label="System Administrator " user-avatar-id="6816f79cc0a8016401c5a33be04be441" ng-click="loadFullProfile(); togglePopover($event); stopPropCheck($event);" role="button" tabindex="0" title="" aria-haspopup="true" class="sub-avatar" ng-style="getBackgroundStyle(users[0] || primary)" style="background-image: url(&quot;a5d3c898c3222010ae17dd981840dd8b.iix?t=small&quot;);" data-original-title=""><!-- ngIf: !users[0].avatar && !groupAvatar --></div></div><!-- ngIf: users.length > 2 && !groupAvatar --><!-- ngIf: users.length > 3 && !groupAvatar --><!-- ngIf: ::presenceEnabled --><sn-presence ng-if="::presenceEnabled" user="users[0].userID || users[0].document" display-name="users[0].displayName" class="ng-scope ng-isolate-scope presence presence-offline"></sn-presence><!-- end ngIf: ::presenceEnabled --></a><!-- end ngIf: !noPopover() --><!-- ngIf: noPopover() --></span>
			<span class="sn-card-component-createdby ng-binding">System Administrator</span></div>

		<div class="sn-card-component sn-card-component_meta">
			<span class="sn-card-component-time" ng-class="{'state-hidden': entry.commentBoxVisible}">
				<div class="datex date-calendar ng-binding">2025-12-21 02:30:57</div>
				<div class="datex date-timeago">
					<sn-time-ago timestamp="entry.sys_created_on" class="ng-isolate-scope"><time data-toggle="tooltip" data-placement="bottom" title="" tabindex="0" data-original-title="12/21/2025, 1:30:57 PM"><span aria-hidden="true" class="ng-binding">2mo ago</span><span class="sr-only ng-binding">2 months ago</span></time></sn-time-ago></div></span>
			<!-- ngIf: ::entry.writable_journal_fields.length --></div>

		<!-- ngIf: entry.commentBoxVisible -->

		<div ng-class="::{'sn-card-component_accent-bar_dark' : entry.entries.journal[0]}" ng-style="::{ 'background-color': fields[entry.entries.journal[0].field_name].color }" class="sn-card-component_accent-bar"></div></div></li>
```
### Acceptance Criteria
- [ ] Clicking the activity icon reveals the panel on the right hand side
- [ ] Clicking the two double right arrows in the activity stream header hides the panel once again

### Constraints
- None

### Resources
- None

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Added Activity Stream side panel that opens/closes via the Activity (pulse) icon in the toolbar. Panel shows mock activity entries with KB number, field changes table, Open Entry button, and author/date metadata. |
| **Commit hash(es)** | `e83131d` |
| **Files touched** | `src/components/knowledge/ActivityStreamPanel.tsx` (NEW), `src/components/knowledge/KnowledgeArticlesTable.tsx`, `src/components/knowledge/KnowledgeArticlesHeader.tsx`, `src/components/layout/ListToolbar.tsx` |
| **Decisions made** | Used conditional render (`activityOpen &&`) rather than CSS transform slide-in for simplicity. Panel is 380px fixed width, table takes remaining space via flex layout. Mock data uses 3 entries matching the reference HTML structure. |
| **Known limitations** | Panel appears/disappears instantly (no slide animation). Data is static mock — no real API integration. |
| **How to verify** | 1. Navigate to /knowledge. 2. Click the Activity (pulse) icon in the toolbar — panel appears on right. 3. Click the >> (ChevronsRight) button in panel header — panel closes. 4. Verify table shrinks/grows when panel opens/closes. |
