
#### Stop Services:

	sudo systemctl stop invertercontrol.service
	sudo systemctl stop consumption_detect.service 

#### Reset Commands:

-*Only Inverter Reset*

	sudo solmatectl control.reset-EVT       

-*Only MPPT Reset*

	sudo solmatectl control.reset-MPP    

-*Only PCM Reset*

	sudo solmatectl control.reset-PCM       

-*Only UI Reset*

	sudo solmatectl control.reset-UI      

-*Reset All*

	sudo solmatectl control.reset-controllers

#### Restart Services:

	sudo systemctl restart invertercontrol.service
	sudo systemctl restart consumption_detect.service 