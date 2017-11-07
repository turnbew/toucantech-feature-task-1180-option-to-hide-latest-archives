TASK DATE: 07.11.2017 - FINISHED: 07.11.2017

TASK SHORT DESCRIPTION: 1180 (Option to hide the 'Latest ...' section for the announcement tab on the front end. Add an enable button beneath current enable/disable toggle. (Text on top of this button: "Display latest content box at top of page:") Add a hover info box to the right side of the new button: "This box pulls in the latest 4 pieces of content from the rest of the page".)

GITHUB REPOSITORY CODE: feature/task-1180-option-to-hide-latest-archives

ORIGINAL WORK: https://github.com/BusinessBecause/network-site/tree/feature/task-1180-option-to-hide-latest-archives

CHANGES
 
	IN FILES: 
	
		\network-site\addons\default\modules\network_settings\language\english\network_settings_lang.php
		
			ADDED CODE: 
			
				$lang['display:latest:archives'] = 'Display latest archives box at top of the page';
			
			
		\network-site\addons\default\modules\network_settings\details.php
		
			ADDED CODE: 
			
				//Add new record to the default_settings table (slug's name: latest_archives_toggle
				if (version_compare($old_version, '2.0.53', 'lt')) 
				{
					$new_record = array(
						'slug' => 'latest_archives_toggle',
						'title' => 'Latest archives toggle',
						'description' => 'This box pulls in the latest 4 pieces of content from the rest of the page',
						'type' => 'radio',
						'default`' => '1',
						'value' => '1',
						'options' => '1=Enabled|0=Disable',
						'is_required' => 0,
						'is_gui' => 1,
						'module' => 'network_settings',
						'order' => '0'
					);

				   $this->db->insert($this->db->dbprefix('settings'), $new_record);
				}		

		\network-site\addons\default\modules\network_settings\views\content\announcements.php
		
			ADDED/CHANGED CODE:
			
				<div class="col-lg-5">
					<table class="table table-condensed setup-questions" style="margin-left: -10px;">
						<?php 
							if(!$enabled_announcement) {
								$btn='btn-success'; 
								$val=1; 
								$btn_val='Enable'; 
								$text_val='disabled';
							}
							else {
								$btn='btn-warning'; 
								$val=0; 
								$btn_val='Disable';
								$text_val='enabled';
							}
						?>
						<tr>
							<td><?=strtoupper($announcement_name);?> is currently <?=$text_val?></td>
						</tr>
						<tr>
							<td>
								<?//=form_checkbox('announcement_toggle', 'announcement_toggle', $enabled, "id='announcement_toggle'") ?>
								<button class="btn <?=$btn?>" id="announcement_toggle" value="<?=$val?>"><?=$btn_val?></button>
							</td>
						</tr>
						<?php 
							//Controlling the latest archives block on the [base_url]/announcements site
							if(!$enabled_latest_archives){
								$btn='btn-success'; 
								$val=1; 
								$btn_val='Enable'; 
								$text_val='disabled';
							}
							else{
								$btn='btn-warning'; 
								$val=0; 
								$btn_val='Disable';
								$text_val='enabled';
							}
						?>
						<tr><td><?php echo lang('display:latest:archives')?></td></tr>
						<tr>
							<td>
								<button class="btn <?=$btn?>" id="latest_archives_toggle" value="<?=$val?>"><?=$btn_val?></button>
								<a class="my-tool-tip" data-toggle="tooltip" data-placement="right" title="<?=$description_latest_archives?>">
									<span class="glyphicon glyphicon-question-sign"></span>	
								</a>	
							</td>
						</tr>
					</table>
				</div>
				
		\network-site\addons\default\modules\network_settings\views\content\announcements.php
		
			ADDED CODE 1:

				//AJAX call from \network_settings\js\content.js file, when you click id = "latest_archives_toggle" button
				public function toggle_latest_archives($value)
				{
					if(!$this->input->is_ajax_request()){
					   echo false;
					   exit;
					}

					$this->load->model('settings/settings_m');

					echo $this->settings_m->update('latest_archives_toggle', array('value' => $value));
				}
				
			ADDED/CHANGED CODE 2:
				
				FROM:
			
					//Deleted
					$enabled = (Settings::get('announcement_toggle')==1);
					
					//CHANGED
					->set('enabled', $enabled) TO: ->set('enabled_announcement', Settings::get('announcement_toggle'))
				
				ADDED: 
					->set('enabled_announcement', Settings::get('announcement_toggle'))
					->set('enabled_latest_archives', Settings::get('latest_archives_toggle'))
					->set('description_latest_archives', Settings::get('latest_archives_toggle', 'description'))
					
		\network-site\addons\default\modules\network_settings\js\content.js
		
		
			ADDED CODE:
			
				$('#latest_archives_toggle').on('click', function(){
					$.ajax({
						type: "POST",
						url: BASE_URI + 'admin-portal/content/toggle_latest_archives/' + $(this).val(),
						success: function (response) {
							if (response == true) location.reload();
						}               
					});
				});
				
		\network-site\addons\default\modules\news\views\view_announcements.php
		
		
			ADDED/CHANGED CODE: 
			
				FROM: 
				
					 <?php if(count($top_stories)): ?>
				
				TO: 
				
					<?php if(count($top_stories) and Settings::get('latest_archives_toggle')) : ?>

				
		\network-site\system\cms\modules\settings\libraries\Settings.php
		
			CHANGED CODE: 
			
				FROM: 
				
					public static function get($name)
					{
						if (isset(self::$cache[$name]))
						{
							return self::$cache[$name];
						}

						$setting = ci()->settings_m->get_by(array('slug' => $name));

						// Setting doesn't exist, maybe it's a config option
						$value = $setting ? $setting->value : config_item($name);

						// Store it for later
						self::$cache[$name] = $value;

						return $value;
					}
					
				TO: 
				
					public static function get($slug_name, $field_name = '')
					{
						if (isset(self::$cache[$slug_name]) and $field_name == '')
						{
							return self::$cache[$slug_name];
						}
							
						$setting = ci()->settings_m->get_by(array('slug' => $slug_name));
						
						if ($field_name == '') 
						{	
							// Setting doesn't exist, maybe it's a config option
							$value = $setting ? $setting->value : config_item($slug_name);
					
							// Store it for later
							self::$cache[$slug_name] = $value;

							return $value;	
						}
						else 
						{
							return $setting->$field_name;
						}
					}
