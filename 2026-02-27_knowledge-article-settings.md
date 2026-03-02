Clicking the settings icon should open this modal

![[Screenshot from 2026-02-28 15-04-20.png]]

**Html**

This is for reference, not 1:1 copy/paste

```html
<div class="modal-content">			<header class="modal-header" aria-label="Personalize List Columns">				<button data-dismiss="GlideModal" aria-label="Close" title="" class="btn btn-icon close icon-cross" id="list_mechanic_closemodal" data-original-title="Close">					<span class="sr-only">Close</span>				</button>			<h2 id="list_mechanic_title" class="modal-title h4">				Personalize List Columns			<button class="btn btn-icon icon-help help">				<span class="sr-only">Help</span>			</button>			</h2>			</header>			<div class="modal-body container-fluid"><rendered_body><script data-description="MessagesTag">(function() {
 var messages = new GwtMessage();
messages.set('Saving');
messages.set('Reloading');
})()</script><form class="list-mechanic" onsubmit="return false;"><div><script eval="true">var slush;
   addLoadEvent(function() {
  slush = new SlushBucket('slush');
   });</script><script eval="true" type="text/javascript">if(navigator.userAgent.match(/iPad/i)) {
var fixIpadMultiselect = function() {
$j('select[multiple]').iosmultiselect();
}
addLoadEvent(function() {
if($j && $j.fn) {
if($j.fn.iosmultiselect){
fixIpadMultiselect();
} else {
var scriptId = "iosMultiselectPlugin";
var scriptTag = document.getElementById(scriptId);
if (!scriptTag) {
var ref = document.getElementsByTagName( "script" )[ 0 ];
var script = document.createElement( "script" );
script.id = scriptId;
script.async = true;
ref.parentNode.insertBefore( script, ref );
script.onload = fixIpadMultiselect;
script.src = 'scripts/lib/jquery.iosmultiselect.js';
}
}
}
});
}</script><div class="container-fluid slushbucket" id="slush"><div class="row form-group"><div class="col-xs-6"><div class="glide-list" role="presentation"><label for="slush_left">Available</label><div class="slushbucket-top"><select aria-label="Available" class="slushselect form-control" id="slush_left" multiple="yes" name="slush_left" onclick="slush.onOptionSelected(event, this)" ondblclick="slush.moveLeftToRight();" onkeydown="slush.onKeyMoveLeftToRight(event); slush.onKeyMoveUpDown(event, this);" onkeyup="slush.onKeyUpMoveUpDown(event, this);" size="10" width="120"><options>--</options><option value="active" aria-selected="false" role="option" title="Active">Active</option><option value="article_id" aria-selected="false" role="option" title="Article ID">Article ID</option><option value="text" aria-selected="false" role="option" title="Article body">Article body</option><option value="article_type" aria-selected="false" role="option" title="Article type">Article type</option><option value="direct" aria-selected="false" role="option" title="Attachment link">Attachment link</option><option value="base_version" aria-selected="false" role="option" title="Base Version">Base Version</option><option value="can_read_user_criteria" aria-selected="false" role="option" title="Can Read">Can Read</option><option value="cannot_read_user_criteria" aria-selected="false" role="option" title="Cannot Read">Cannot Read</option><option value="category" aria-selected="false" role="option" title="Category(category)">Category(category)</option><option value="sys_class_name" aria-selected="false" role="option" title="Class">Class</option><option value="cmdb_ci" aria-selected="false" role="option" title="Configuration item">Configuration item</option><option value="sys_created_on" aria-selected="false" role="option" title="Created">Created</option><option value="sys_created_by" aria-selected="false" role="option" title="Created by">Created by</option><option value="description" aria-selected="false" role="option" title="Description">Description</option><option value="disable_commenting" aria-selected="false" role="option" title="Disable commenting">Disable commenting</option><option value="disable_suggesting" aria-selected="false" role="option" title="Disable suggesting">Disable suggesting</option><option value="display_attachments" aria-selected="false" role="option" title="Display attachments">Display attachments</option><option value="display_number" aria-selected="false" role="option" title="Display number">Display number</option><option value="sys_domain" aria-selected="false" role="option" title="Domain">Domain</option><option value="sys_domain_path" aria-selected="false" role="option" title="Domain Path">Domain Path</option><option value="editor_type" aria-selected="false" role="option" title="Editor Type">Editor Type</option><option value="flagged" aria-selected="false" role="option" title="Flagged">Flagged</option><option value="generated_with_now_assist" aria-selected="false" role="option" title="Generated With Now Assist">Generated With Now Assist</option><option value="helpful_count" aria-selected="false" role="option" title="Helpful count">Helpful count</option><option value="image" aria-selected="false" role="option" title="Image">Image</option><option value="instrumentation_metadata" aria-selected="false" role="option" title="Instrumentation metadata">Instrumentation metadata</option><option value="kb_knowledge_base" aria-selected="false" role="option" title="Knowledge base">Knowledge base</option><option value="latest" aria-selected="false" role="option" title="Latest">Latest</option><option value="latest_aqi" aria-selected="false" role="option" title="Latest AQI">Latest AQI</option><option value="meta" aria-selected="false" role="option" title="Meta">Meta</option><option value="meta_description" aria-selected="false" role="option" title="Meta Description">Meta Description</option><option value="ownership_group" aria-selected="false" role="option" title="Ownership Group">Ownership Group</option><option value="published" aria-selected="false" role="option" title="Published">Published</option><option value="rating" aria-selected="false" role="option" title="Rating">Rating</option><option value="replacement_article" aria-selected="false" role="option" title="Replacement Article">Replacement Article</option><option value="retired" aria-selected="false" role="option" title="Retired">Retired</option><option value="revised_by" aria-selected="false" role="option" title="Revised By">Revised By</option><option value="roles" aria-selected="false" role="option" title="Roles">Roles</option><option value="scheduled_publish_date" aria-selected="false" role="option" title="Scheduled publish date">Scheduled publish date</option><option value="source" aria-selected="false" role="option" title="Source task">Source task</option><option value="summary" aria-selected="false" role="option" title="Summary">Summary</option><option value="sys_tags" aria-selected="false" role="option" title="Tags">Tags</option><option value="taxonomy_topic" aria-selected="false" role="option" title="Taxonomy topic">Taxonomy topic</option><option value="topic" aria-selected="false" role="option" title="Topic">Topic</option><option value="sys_updated_by" aria-selected="false" role="option" title="Updated by">Updated by</option><option value="sys_mod_count" aria-selected="false" role="option" title="Updates">Updates</option><option value="use_count" aria-selected="false" role="option" title="Use count">Use count</option><option value="valid_to" aria-selected="false" role="option" title="Valid to">Valid to</option><option value="view_as_allowed" aria-selected="false" role="option" title="View as allowed">View as allowed</option><option value="sys_view_count" aria-selected="false" role="option" title="View count">View count</option><option value="wiki" aria-selected="false" role="option" title="Wiki">Wiki</option></select><div class="button-column" id="addRemoveButtons"><a class="btn btn-default icon-chevron-right slush-bucket-add" href="javascript:void(0)" onclick="slush.moveLeftToRight();" onkeydown="slush.keydownMoveLeftToRight(event)" role="button" title="" data-original-title="Add"><span class="sr-only">Add selected options to the Selected listbox</span></a><a class="btn btn-default icon-chevron-left slush-bucket-remove" href="javascript:void(0)" onclick="slush.moveRightToLeft();" onkeydown="slush.keydownMoveRightToLeft(event);" role="button" title="Remove"><span class="sr-only">Remove selected options from the Selected listbox</span></a></div></div></div></div><div class="col-xs-6"><div class="glide-list" role="presentation"><label for="slush_right">Selected</label><div class="slushbucket-top slushbody"><select aria-label="Selected" class="slushselect form-control" id="slush_right" multiple="yes" name="slush_right" onclick="slush.onOptionSelected(event, this)" ondblclick="slush.moveRightToLeft();" onkeydown="slush.onKeyMoveRightToLeft(event); slush.onKeyMoveUpDown(event, this);" onkeyup="slush.onKeyUpMoveUpDown(event, this);" size="10" width="120"><option value="number" aria-selected="false" role="option" title="Number">Number</option><option value="version" aria-selected="false" role="option" title="Version">Version</option><option value="short_description" aria-selected="false" role="option" title="Short description">Short description</option><option value="author" aria-selected="false" role="option" title="Author">Author</option><option value="kb_category" aria-selected="false" role="option" title="Category(kb_category)">Category(kb_category)</option><option value="workflow_state" aria-selected="false" role="option" title="Workflow">Workflow</option><option value="sys_updated_on" aria-selected="false" role="option" title="Updated">Updated</option></select><div class="button-column" id="upDownButtons"><a class="btn btn-default icon-chevron-up" href="javascript:void(0)" onclick="slush.moveUp();" ondblclick="slush.moveUp();" role="button" title="Move up"><span class="sr-only">Move selected options up in the Selected listbox</span></a><a class="btn btn-default icon-chevron-down" href="javascript:void(0)" onclick="slush.moveDown();" ondblclick="slush.moveDown();" role="button" title="Move down"><span class="sr-only">Move selected options down in the Selected listbox</span></a></div></div></div></div></div></div></div><div><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.wrap" name="ni.wrap" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.wrap">Wrap column text</label><input class="checkbox_list_mechanic" id="wrap" name="wrap" type="hidden"></span><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.compact" name="ni.compact" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.compact">Compact rows</label><input class="checkbox_list_mechanic" id="compact" name="compact" type="hidden"></span><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.highlighting" name="ni.highlighting" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.highlighting">Active row highlighting</label><input class="checkbox_list_mechanic" id="highlighting" name="highlighting" type="hidden"></span><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.field_style_circles" name="ni.field_style_circles" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.field_style_circles">Modern cell coloring</label><input class="checkbox_list_mechanic" id="field_style_circles" name="field_style_circles" type="hidden"></span></div><div><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.list_edit_enable" name="ni.list_edit_enable" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.list_edit_enable">Enable list edit</label><input class="checkbox_list_mechanic" id="list_edit_enable" name="list_edit_enable" type="hidden"></span><span class="input-group-checkbox"><input class="checkbox_list_mechanic checkbox" id="ni.list_edit_double" name="ni.list_edit_double" onclick="setCheckBox(this);" type="checkbox" value="false"><label class="checkbox-label" for="ni.list_edit_double">Double click to edit</label><input class="checkbox_list_mechanic" id="list_edit_double" name="list_edit_double" type="hidden"></span></div><div class="modal-footer"><span class="pull-left status_msg_update"><button class="btn btn-default" disabled="" id="reset_column_defaults" onclick="actionOK(true)" type="submit">Reset to column defaults</button></span><span class="pull-right"><button class="btn btn-default" id="cancel_button" onclick="(window.GlideFrame ? window.GlideFrame.close() : (window.GlideDialogWindow || window.GlideModalForm).prototype.locate(this).destroy()); return false" style="min-width: 5em;" title="" type="submit">Cancel</button>&nbsp;
<button class="btn btn-primary" id="ok_button" onclick="actionOK()" style="min-width: 5em;" title="" type="submit" data-original-title="">OK</button></span></div><script eval="true">$j(".list-mechanic").keydown(function(evt) {
    // DEF0125399: if a user were to press Enter on a non-button, non-link element it would unintentionally submit the form
if (evt.keyCode === 13 && !["BUTTON", "A"].includes(evt.target.tagName)) {
evt.preventDefault();
}
});</script></form><script eval="true">var LM = 'gs.include("ListMechanic"); var v = new ListMechanic();';
function actionOK(reset) {
  var keys = ["Saving", "Reloading"];
  var msgs = new GwtMessage().getMessages(keys);
  var array = slush.getValues(slush.getRightSelect());  var f = array.join(',');
  var o = slush.getRightValues();
  var changes = true;
  if (f == o)
     changes = false;
  setStatus(msgs["Saving"] + "...");
  var enableCompactList = gel('ni.compact').checked;
  var enableHighlighting = gel('ni.highlighting').checked;
  var enableTextWrap = gel('ni.wrap').checked;
  var enableModernCellColoring = gel('ni.field_style_circles').checked;
  var ga = new GlideAjax('UIPage');
  ga.addParam('sysparm_name','createListMechanic');
  ga.addParam('sysparm_reset',reset ? true : false);
  ga.addParam('sysparm_list_view',g_list_view);
  ga.addParam('sysparm_list_parent',g_list_parent);
  ga.addParam('sysparm_list_parent_id',g_list_parent_id);
  ga.addParam('sysparm_list_relationship',g_list_relationship);
  ga.addParam('sysparm_table',g_table);
  ga.addParam('sysparm_f',f);
  ga.addParam('sysparm_changes',changes);
  ga.addParam('sysparm_compact',enableCompactList);
  ga.addParam('sysparm_wrap',enableTextWrap);
  ga.addParam('sysparm_highlighting',enableHighlighting);
  ga.addParam('sysparm_field_style_circles', enableModernCellColoring);
  var enableListEdit = gel('ni.list_edit_enable');
  if (enableListEdit)
     ga.addParam('sysparm_edit_enable', enableListEdit.checked);
  var enableDblClickToEdit = gel('ni.list_edit_double');
  if (enableDblClickToEdit)
     ga.addParam('sysparm_edit_double',enableDblClickToEdit.checked);
  trackMetrics({
"interface-type": 'CoreUI',
"table": g_table,
"parentId": g_list_parent_id,
"listParent": g_list_parent,
"relationship": g_list_relationship,
"columns": f,
"hasChanges": changes,
"enableListEdit": enableListEdit && enableListEdit.checked,
"enableCompactList": enableCompactList,
"enableDoubleClickToEdit": enableDblClickToEdit && enableDblClickToEdit.checked,
"enableHighlighting": enableHighlighting,
"enableTextWrap": enableTextWrap,
"enableModernCellColoring": enableModernCellColoring
  });
  ga.getXMLWait();
  ga.getAnswer();
  setStatus("&lt;span style='color:green'&gt; " + msgs['Reloading'] + "&lt;/span&gt;");
  setTimeout('reloadWindow(self, true);', 1);
  GlideModal.prototype.get('list_mechanic').destroy();
  return false;
}
function setStatus(statusHTML) {
  gel('list_mechanic').getElementsByClassName('status_msg_update')[0].innnerHTML = statusHTML;
}
function getList() {
  if (typeof g_table == "undefined")
    g_table = 'sys_user';
  if (typeof g_list_view == "undefined")
    g_list_view = "";
  if (typeof g_list_parent == "undefined")
    g_list_parent = "";
  if (typeof g_list_relationship == "undefined")
    g_list_relationship = "";
  var ga = new GlideAjax('UIPage');
  ga.addParam('sysparm_name','getList');
  ga.addParam('sysparm_list_view',g_list_view);
  ga.addParam('sysparm_list_parent',g_list_parent);
  ga.addParam('sysparm_list_parent_id',g_list_parent_id);
  ga.addParam('sysparm_list_relationship',g_list_relationship);
  ga.addParam('sysparm_table',g_table);
  ga.getXML(listResponse);
}
function listResponse(request) {
  var xml = request.responseXML.documentElement;
  var z = xml.getElementsByTagName("columns")[0];
  populateSelect(gel('slush_left'), z);
  z = xml.getElementsByTagName("selected")[0];
  populateSelect(gel('slush_right'), z);
  var array = slush.getValues(slush.getRightSelect());
  var r = array.join(",");
  slush.saveRightValues(r);
  // a user list?
  z = xml.getElementsByTagName("choice_list_set")[0];
  var a = z.getAttribute("user_list");
  gel('ni.highlighting').checked = eval(z.getAttribute('table.highlighting'));
  gel('ni.compact').checked = eval(z.getAttribute('table.compact'));
  gel('ni.wrap').checked = eval(z.getAttribute('table.wrap'));
  gel('ni.field_style_circles').checked = eval(z.getAttribute('field_style_circles'));
  var o = gel('ni.list_edit_enable');
  if (o)
    o.checked = eval(z.getAttribute('list_edit_enable'));
  var o = gel('ni.list_edit_double');
  if (o)
    o.checked = eval(z.getAttribute('list_edit_double'));
  if (a == 'true') {
     gel('reset_column_defaults').disabled = false;
  } else {
    gel('reset_column_defaults').disabled = true;
  }
}
function populateSelect(select, xml) {
  select.options.length = 0;
  var choices = xml.getElementsByTagName("choice");
  for (var i = 0; i &lt; choices.length; i++ ) {
    var choice = choices[i];
    var o = new Option(choice.getAttribute('label'), choice.getAttribute('value'));
o.setAttribute('aria-selected', false);
o.setAttribute('role', 'option');
o.setAttribute('title', choice.getAttribute('label'));
    select.options[select.options.length] = o;
  }
}
function trackMetrics(payload) {
if (!GlideUXMetrics)
return;
GlideUXMetrics.track('Personalize List Columns', payload);
}
getList();</script></rendered_body></div>		</div>
```