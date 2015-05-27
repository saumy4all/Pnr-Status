# Pnr-Status
Project to get the details of trains, seat availability, ticket booking, PNR status 
package com.example.railpnr;


import android.os.Bundle;
import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.AlertDialog.Builder;
import android.app.FragmentManager;
import android.app.FragmentTransaction;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.res.Configuration;
import android.telephony.SmsManager;
import android.view.KeyEvent;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.animation.AnimationUtils;
import android.widget.Button;
import android.widget.EditText;
import android.widget.PopupMenu;
import android.widget.RatingBar;
import android.widget.TextView;
import android.widget.Toast;

@SuppressLint("NewApi") 
public class MainActivity extends Activity implements OnClickListener
{
	TextView title,service;
	Button pnrck,booktk;
	int fragCheck;
	FragmentManager fragmentManager=getFragmentManager();
    FragmentTransaction fragmentTransaction=fragmentManager.beginTransaction();
	@Override
    protected void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        pnrck=(Button)findViewById(R.id.pnr);
        pnrck.setOnClickListener(this);
        booktk=(Button)findViewById(R.id.book);
        booktk.setOnClickListener(this);
        title=(TextView)findViewById(R.id.txt1);
        title.setOnClickListener(this);
        service=(TextView)findViewById(R.id.smsService);
    	title.startAnimation(AnimationUtils.loadAnimation(MainActivity.this, R.anim.left));
    	
        
        service.setOnClickListener(new OnClickListener() //pop up menu
        {
			
			@Override
			public void onClick(View v) 
			{
				PopupMenu popup=new PopupMenu(MainActivity.this,service );
				popup.getMenuInflater().inflate(R.menu.sms_popup, popup.getMenu());
				popup.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() 
				{
					
					@Override
					public boolean onMenuItemClick(MenuItem v) 
					{
						if(v.getItemId()==R.id.item1)
						{
							Intent i=new Intent(getApplicationContext(), JsonCheck.class);
							i.putExtra("off","22");
							startActivity(i);
							fragCheck=0;//for pnr by sms
						}
						if(v.getItemId()==R.id.item2)
						{
							booktk.setVisibility(View.INVISIBLE);
			    			pnrck.setVisibility(View.INVISIBLE);
			    			service.setVisibility(View.INVISIBLE);
							OfflineCheck layout=new OfflineCheck();
			    			fragmentTransaction.replace(android.R.id.content, layout);
			    			fragmentTransaction.commit();
							fragCheck=1;//for calling 139 
						}
						if(v.getItemId()==R.id.item3)
						{
							Intent i=new Intent(getApplicationContext(), TrainNumber.class);
							i.putExtra("off","22");
							startActivity(i);
							fragCheck=2;//for train schedule
						}
						return false;
					}
				});
				popup.show();
			}
		});        
    }
    
    @Override
    public boolean onCreateOptionsMenu(Menu menu) 
    {
    	
    	super.onCreateOptionsMenu(menu);
    	
    	// Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

	@Override
	public boolean onOptionsItemSelected(MenuItem item)//Option menu
    {
		
		Intent i;
    	switch(item.getItemId())
    	{
    	    case R.id.pnrstat://ok
    	    	i=new Intent(getApplicationContext(), JsonCheck.class);
    	    	i.putExtra("off","00");
    			startActivity(i);
    	    	   return true;	  
    	    	   
    		case R.id.booktkt://ok
    			i=new Intent(getApplicationContext(), Browser.class);
    			i.putExtra("ID", "2");
    			startActivity(i);
    			return true;
    		
    		case R.id.stations://working
    			i=new Intent(getApplicationContext(), StationAvailability.class);
    			i.putExtra("ID", "2");//for display
    			startActivity(i);
    		
    			
    			return true;
    		
    		case R.id.status://ok
    			i=new Intent(getApplicationContext(), Browser.class);
    			i.putExtra("ID", "4");
    			startActivity(i);
        		return true;
    		
    		case R.id.Seat://ok,Seat layout
    			FragmentManager fragmentManager=getFragmentManager();
    		    FragmentTransaction fragmentTransaction=fragmentManager.beginTransaction();
    			SeatLayout layout=new SeatLayout();
    			fragmentTransaction.replace(android.R.id.content, layout);
    			booktk.setVisibility(View.INVISIBLE);
    			pnrck.setVisibility(View.INVISIBLE);
    			service.setVisibility(View.INVISIBLE);
    			fragmentTransaction.commit();
    			
    			return true;
    		
    		case R.id.available://ok, seat availability
    			i=new Intent(getApplicationContext(), StationAvailability.class);
    			i.putExtra("ID", "1");//for dispaly
    			startActivity(i);
    			return true;
    			
    		case R.id.schedule://ok, train enquary
    			i=new Intent(getApplicationContext(),TrainNumber.class);
    			i.putExtra("off","11");
    			startActivity(i);
    			return true;
    		
    		case R.id.rate://need editing
    			rateDialogue();
    			return true;
    		default:return super.onOptionsItemSelected(item);
    	}
    	
    }
	private void rateDialogue() 
	{
		final AlertDialog.Builder builder=new AlertDialog.Builder(this);
		final RatingBar rate=new RatingBar(this);
		rate.setNumStars(5);
		builder.setTitle("Rate Us !!").setIcon(R.drawable.star).setMessage("If you liked the app kindly rate us");
		builder.setView(rate);
		builder.setPositiveButton("OK", new DialogInterface.OnClickListener() 
		{
			
			@Override
			public void onClick(DialogInterface arg0, int arg1) 
			{
				Toast.makeText(getApplicationContext(), String.valueOf(rate.getProgress())+" rating given", Toast.LENGTH_LONG).show();				
				if(rate.getProgress()<4)
				{
					feedback();
				}
			}

		})
		.setNegativeButton("CANCEL", new DialogInterface.OnClickListener() 
		{
			
			@Override
			public void onClick(DialogInterface dialog, int which) {
				// TODO Auto-generated method stub
				dialog.cancel();
				
			}
		});
		builder.create();
		builder.show();
	}
	private void feedback() 
	{
		AlertDialog.Builder  build=new AlertDialog.Builder(this);
		final EditText txt=new EditText(this);
		txt.setMinLines(3);
		build.setTitle("Feedback!!").setView(txt)
		.setPositiveButton("ok", new DialogInterface.OnClickListener() 
		{
			
			@Override
			public void onClick(DialogInterface dialog, int which) 
			{
				Toast.makeText(getApplicationContext(),txt.getText().toString()+" As FeedBack" , Toast.LENGTH_LONG).show();
				
			}
		})
		.setCancelable(true);
		build.create();
		build.show();
	}

	public void broadcastsms(String phoneNumber,String smsBody,Context contexxxt)
	{
		String SMS_SENT = "SMS_SENT";
		String SMS_DELIVERED = "SMS_DELIVERED";
	 
		PendingIntent sentPendingIntent = PendingIntent.getBroadcast(contexxxt, 0, new Intent(SMS_SENT), 0);
		PendingIntent deliveredPendingIntent = PendingIntent.getBroadcast(contexxxt, 0, new Intent(SMS_DELIVERED), 0);
	 
		// For when the SMS has been sent
		registerReceiver(new BroadcastReceiver() 
		{
			@Override
			public void onReceive(Context context, Intent intent) 
			{
				switch (getResultCode()) 
				{
	            	case Activity.RESULT_OK:
	            		Toast.makeText(context, "SMS sent successfully", Toast.LENGTH_SHORT).show();
	            		break;
	            	case SmsManager.RESULT_ERROR_GENERIC_FAILURE:
	            		Toast.makeText(context, "Generic failure cause", Toast.LENGTH_SHORT).show();
	            		break;
	            	case SmsManager.RESULT_ERROR_NO_SERVICE:
	            		Toast.makeText(context, "Service is currently unavailable", Toast.LENGTH_SHORT).show();
	            		break;
	            	case SmsManager.RESULT_ERROR_NULL_PDU:
	            		Toast.makeText(context, "No pdu provided", Toast.LENGTH_SHORT).show();
	            		break;
	            	case SmsManager.RESULT_ERROR_RADIO_OFF:
	            		Toast.makeText(context, "Radio was explicitly turned off", Toast.LENGTH_SHORT).show();
	            		break;
				}
			}
		}, new IntentFilter(SMS_SENT));
	 
		// For when the SMS has been delivered
		registerReceiver(new BroadcastReceiver() 
		{
			@Override
			public void onReceive(Context context, Intent intent) 
			{
				switch (getResultCode()) 
				{
	            	case Activity.RESULT_OK:
	            		Toast.makeText(getBaseContext(), "SMS delivered", Toast.LENGTH_SHORT).show();
	            		break;
	            	case Activity.RESULT_CANCELED:
	            		Toast.makeText(getBaseContext(), "SMS not delivered", Toast.LENGTH_SHORT).show();
	            		break;
				}
			}
		}, new IntentFilter(SMS_DELIVERED));
	 
		// Get the default instance of SmsManager
		SmsManager smsManager = SmsManager.getDefault();
		// Send a text based SMS
		smsManager.sendTextMessage(phoneNumber, null, smsBody, sentPendingIntent, deliveredPendingIntent);
	}
	@Override
	public void onClick(View v) 
	{
		if(v.getId()==R.id.pnr)
		{
			Intent i=new Intent(getApplicationContext(), JsonCheck.class);
			i.putExtra("off", "00");
			startActivity(i);
		}
		if(v.getId()==R.id.book)
		{
			Intent i=new Intent(getApplicationContext(), Browser.class);
			i.putExtra("ID", "2");
			startActivity(i);
		}
		
	}
	public boolean onKeyDown(int keyCode, KeyEvent event) //for dialog before exiting
	{
	    if (keyCode == KeyEvent.KEYCODE_BACK) 
	    {
	        exitByBackKey();
	        //moveTaskToBack(false);
	        return true;
	    }
	    return super.onKeyDown(keyCode, event);
	}

	protected void exitByBackKey() 
	{
	    AlertDialog alertbox = new AlertDialog.Builder(this)
	    .setMessage("Do you want to exit application?")
	    .setPositiveButton("Yes", new DialogInterface.OnClickListener() 
	    {
	        // do something when the button is clicked
	        public void onClick(DialogInterface arg0, int arg1) 
	        {
	            finish();
	            //close();
	        }
	    })
	    .setNegativeButton("No", new DialogInterface.OnClickListener() 
	    {
	        // do something when the button is clicked
	        public void onClick(DialogInterface arg0, int arg1) 
	        {
	        }
	    }).show();
	}
}
