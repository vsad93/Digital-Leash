package com.example.srinivasvaddadi.digitalleash;

import android.content.Intent;
import android.location.Location;
import android.os.AsyncTask;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    Button statusButton,createButton,updateButton;
    EditText userName,latitude,longitude,radius;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        userName = (EditText)findViewById(R.id.editText) ;
        radius = (EditText)findViewById(R.id.editText2) ;
        latitude = (EditText)findViewById(R.id.editText5) ;
        longitude = (EditText)findViewById(R.id.editText3) ;
        createButton = (Button)findViewById(R.id.button);
        statusButton = (Button)findViewById(R.id.button3);

        updateButton = (Button)findViewById(R.id.button2);
        updateButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String url =  "https://turntotech.firebaseio.com/digitalleash/"+userName.getText().toString()+".json";
                new PatchData().execute(url);
            }
        });

        statusButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                //startActivity(new Intent(MainActivity.this,Main2Activity.class));
                String url =  "https://turntotech.firebaseio.com/digitalleash/"+userName.getText().toString()+".json";
                new GetData().execute(url);
            }
        });


        createButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String url =  "https://turntotech.firebaseio.com/digitalleash/"+userName.getText().toString()+".json";
                new PutData().execute(url);
            }
        });


    }


    public class PutData extends AsyncTask<String, String, String> {

        HttpURLConnection urlConnection;
        String json = null;

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            JSONObject jsonObject = new JSONObject();
            try {
                jsonObject.put("username",userName.getText().toString());
                jsonObject.put("latitude", latitude.getText().toString());
                jsonObject.put("longitude", longitude.getText().toString());
                jsonObject.put("radius",radius.getText().toString());
                json = jsonObject.toString();
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
        @Override
        protected String doInBackground(String... args) {



            try {
                URL url = new URL(args[0]);
                urlConnection = (HttpURLConnection) url.openConnection();
                urlConnection.setRequestMethod("PUT"); // PATCH
                urlConnection.setDoOutput(true);
                urlConnection.setRequestProperty("Content-Type","application/json");
                urlConnection.setRequestProperty("Accept","application/json");
                OutputStreamWriter outputStreamWriter = new OutputStreamWriter(urlConnection.getOutputStream());
                outputStreamWriter.write(json);
                outputStreamWriter.flush();
                outputStreamWriter.close();
                urlConnection.getResponseCode();
                urlConnection.disconnect();

            } catch (Exception e) {
                e.printStackTrace();
            }

            return null;

        }

        @Override
        protected void onPostExecute(String result) {

        }

    }
    public class PatchData extends AsyncTask<String, String, String> {

        HttpURLConnection urlConnection;
        String json = null;

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            JSONObject jsonObject = new JSONObject();
            try {
                jsonObject.put("username",userName.getText().toString());
                jsonObject.put("latitude", latitude.getText().toString());
                jsonObject.put("longitude", longitude.getText().toString());
                jsonObject.put("radius",radius.getText().toString());
                json = jsonObject.toString();
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }

        @Override
        protected String doInBackground(String... args) {
            try {
                URL url = new URL(args[0]);
                urlConnection = (HttpURLConnection) url.openConnection();
                urlConnection.setRequestMethod("PATCH");
                urlConnection.setDoOutput(true);
                urlConnection.setRequestProperty("Content-Type","application/json");
                urlConnection.setRequestProperty("Accept","application/json");
                OutputStreamWriter outputStreamWriter = new OutputStreamWriter(urlConnection.getOutputStream());
                outputStreamWriter.write(json);
                outputStreamWriter.flush();
                outputStreamWriter.close();
                urlConnection.getResponseCode();
                urlConnection.disconnect();

            } catch (Exception e) {
                e.printStackTrace();
            }

            return null;

        }

    }

    public class GetData extends AsyncTask<String, String, String> {

        HttpURLConnection urlConnection;

        @Override
        protected String doInBackground(String... args) {

            StringBuilder result = new StringBuilder();

            try {
                URL url = new URL(args[0]);
                urlConnection = (HttpURLConnection) url.openConnection();
                InputStream in = new BufferedInputStream(urlConnection.getInputStream());

                BufferedReader reader = new BufferedReader(new InputStreamReader(in));

                String line;
                while ((line = reader.readLine()) != null) {
                    result.append(line);
                }

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                urlConnection.disconnect();
            }


            return result.toString();
        }

        @Override
        protected void onPostExecute(String result) {

            // parse your result and get all your data
            Log.e("JSON Return",result);
            try {
                JSONObject jsonObject = new JSONObject(result);
                String parent_latitude = jsonObject.getString("latitude");
                String parent_longitude = jsonObject.getString("longitude");
                String child_latitude = jsonObject.getString("child_latitude");
                String child_longitude = jsonObject.getString("child_longitude");
                double distanceRadius = jsonObject.getDouble("radius");

                Location parentLoc = new Location("");
                parentLoc.setLatitude(Double.parseDouble(parent_latitude));
                parentLoc.setLongitude(Double.parseDouble(parent_longitude));

                Location childLoc = new Location("");
                childLoc.setLatitude(Double.parseDouble(child_latitude));
                childLoc.setLongitude(Double.parseDouble(child_longitude));

                // Find distance
                double actualDistance = parentLoc.distanceTo(childLoc);

                if(actualDistance>distanceRadius){
                    // Child is Out Of Zone
                    startActivity(new Intent(MainActivity.this,Main2Activity.class));
                }else{
                    // Child is in Zone
                    startActivity(new Intent(MainActivity.this,Main3Activity.class));
                }

                } catch (JSONException e) {
                e.printStackTrace();
            }
        }



    }

}
