# wsecurity
aplicacion para evitar la inseguridad contra las mujeres con botón de pánico y llamadas falas para asegurar la vida y seguridad de la persona en riesgo  
codigo de android studio:



package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.annotation.SuppressLint;
import android.os.Bundle;
// 01 ----------------------------------------------------------------------------------------------
import android.view.View;
import android.widget.Button;
import android.widget.Spinner;
import android.widget.Toast;
import androidx.activity.result.ActivityResult;
import androidx.activity.result.ActivityResultCallback;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import android.Manifest;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.bluetooth.BluetoothSocket;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.util.Log;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.TextView;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.ArrayList;
import java.util.Set;
import java.util.UUID;
import com.example.myapplication.R;

// 01 ----------------------------------------------------------------------------------------------
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private static final UUID BT_MODULE_UUID = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
    private static final int REQUEST_ENABLE_BT = 1;
    private static final int REQUEST_BLUETOOTH_CONNECT_PERMISSION = 3;
    private static final int REQUEST_FINE_LOCATION_PERMISSION = 2;
    private BluetoothAdapter mBtAdapter;
    private BluetoothSocket btSocket;
    private BluetoothDevice DispositivoSeleccionado;
    private ConnectedThread MyConexionBT;
    private ArrayList<String> mNameDevices = new ArrayList<>();
    private ArrayAdapter<String> deviceAdapter;

    // 02 ------------------------------------------------------------------------------------------
    Button IdBtnBuscar,IdBtnConectar,IdBtnLuz1on,IdBtnLuz1off,IdBtnLuz2on,IdBtnLuz2off,IdBtnDesconectar;
    Spinner IdDisEncontrados;
    // 02 ------------------------------------------------------------------------------------------
    @SuppressLint("MissingInflatedId")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // 03 --------------------------------------------------------------------------------------
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        requestBluetoothConnectPermission();
        requestLocationPermission();


        IdBtnBuscar = findViewById(R.id.IdBtnBuscar);
        IdBtnConectar = findViewById(R.id.IdBtnConectar);
        IdBtnLuz1on = findViewById(R.id.IdBtnLuz1on);
        IdBtnLuz1off = findViewById(R.id.IdBtnLuz1off);
        IdBtnConectar = findViewById(R.id.IdBtnConectar);
        IdBtnDesconectar = findViewById(R.id.IdBtnDesconectar);
        IdDisEncontrados = findViewById(R.id.IdDisEncontrados);

        deviceAdapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, mNameDevices);
        deviceAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        IdDisEncontrados.setAdapter(deviceAdapter);


        IdBtnBuscar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                DispositivosVinculados();
            }
        });

        IdBtnConectar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                ConectarDispBT();
            }
        });

        IdBtnDesconectar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                    // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                    if (btSocket!=null)
                    {
                        try {btSocket.close();}
                        catch (IOException e)
                        { Toast.makeText(getBaseContext(), "Error", Toast.LENGTH_SHORT).show();}
                    }
                    finish();


            }
        });

        IdBtnLuz1on.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                Toast.makeText(getBaseContext(),"SE HA PRESIONADO EL BOTON LUZ 1 ON",Toast.LENGTH_SHORT).show();
                MyConexionBT.write('a');

            }
        });

        IdBtnLuz1off.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // CODIGO A EJECUTAR DE ESTE EVENTO PARA ESTE BOTON
                MyConexionBT.write('b');
            }
        });

        // 03 --------------------------------------------------------------------------------------
    }
    // 04 ------------------------------------------------------------------------------------------
    ActivityResultLauncher<Intent> someActivityResultLauncher = registerForActivityResult(
            new ActivityResultContracts.StartActivityForResult(),
            new ActivityResultCallback<ActivityResult>() {
                @Override
                public void onActivityResult(ActivityResult result) {

                    if (result.getResultCode() == MainActivity.REQUEST_ENABLE_BT) {
                        Log.d(TAG, "ACTIVIDAD REGISTRADA");
                        //Toast.makeText(getBaseContext(), "ACTIVIDAD REGISTRADA", Toast.LENGTH_SHORT).show();
                    }
                }
            });

    public void DispositivosVinculados() {
        mBtAdapter = BluetoothAdapter.getDefaultAdapter();
        if (mBtAdapter == null) {
            showToast("Bluetooth no disponible en este dispositivo.");
            finish();
            return;
        }

        if (!mBtAdapter.isEnabled()) {
            Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
                return;
            }
            someActivityResultLauncher.launch(enableBtIntent);
        }

        IdDisEncontrados.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parentView, View selectedItemView, int position, long id) {
                DispositivoSeleccionado = getBluetoothDeviceByName(mNameDevices.get(position));
            }

            @Override
            public void onNothingSelected(AdapterView<?> parentView) {
                DispositivoSeleccionado = null;
            }
        });

        Set<BluetoothDevice> pairedDevices = mBtAdapter.getBondedDevices();
        if (pairedDevices.size() > 0) {
            for (BluetoothDevice device : pairedDevices) {
                mNameDevices.add(device.getName());
            }
            deviceAdapter.notifyDataSetChanged();
        } else {
            showToast("No hay dispositivos Bluetooth emparejados.");
        }
    }

    // Agrega este método para solicitar el permiso
    private void requestLocationPermission() {
        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, REQUEST_FINE_LOCATION_PERMISSION);
    }

    // Agrega este método para solicitar el permiso
    private void requestBluetoothConnectPermission() {
        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.BLUETOOTH_CONNECT}, REQUEST_BLUETOOTH_CONNECT_PERMISSION);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if (requestCode == REQUEST_BLUETOOTH_CONNECT_PERMISSION) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Log.d(TAG, "Permiso concedido, ahora puedes utilizar funciones de Bluetooth que requieran BLUETOOTH_CONNECT");
            } else {
                Log.d(TAG, "Permiso denegado, debes manejar este caso según tus necesidades");
            }
        }
    }

    private BluetoothDevice getBluetoothDeviceByName(String name) {

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
            Log.d(TAG, " ----->>>>> ActivityCompat.checkSelfPermission");
        }
        Set<BluetoothDevice> pairedDevices = mBtAdapter.getBondedDevices();
        for (BluetoothDevice device : pairedDevices) {
            if (device.getName().equals(name)) {
                return device;
            }
        }
        return null;
    }
    private void ConectarDispBT() {
        if (DispositivoSeleccionado == null) {
            showToast("Selecciona un dispositivo Bluetooth.");
            return;
        }

        try {
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
                return;
            }
            btSocket = DispositivoSeleccionado.createRfcommSocketToServiceRecord(BT_MODULE_UUID);
            btSocket.connect();
            MyConexionBT = new ConnectedThread(btSocket);
            MyConexionBT.start();
            showToast("Conexión exitosa.");
        } catch (IOException e) {
            showToast("Error al conectar con el dispositivo.");
        }
    }

    private class ConnectedThread extends Thread {
        private final OutputStream mmOutStream;
        ConnectedThread(BluetoothSocket socket) {
            InputStream tmpIn = null;
            OutputStream tmpOut = null;

            try {
                tmpIn = socket.getInputStream();
                tmpOut = socket.getOutputStream();
            } catch (IOException e) {
                showToast("Error al crear el flujo de datos.");
            }

            mmOutStream = tmpOut;
        }
        public void write(char input) {
            //byte msgBuffer = (byte)input;
            try {
                mmOutStream.write((byte)input);
            } catch (IOException e) {
                Toast.makeText(getBaseContext(), "La Conexión fallo", Toast.LENGTH_LONG).show();
                finish();
            }
        }
    }
    private void showToast(final String message) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(getApplicationContext(), message, Toast.LENGTH_SHORT).show();
            }
        });
    }
 
    // 04 ------------------------------------------------------------------------------------------

codigo de arduino para modulo bluetooth:

#include <SoftwareSerial.h> //Quitar los espacios de esta parte  < SoftwareSerial.h >

SoftwareSerial BT(10,11); // 10 RX, 11 TX.

byte BUZZER = 3;


char CharIN = ' ';

void setup()  
{
  Serial.begin(9600); 
  BT.begin(9600); 
  
  pinMode(BUZZER, OUTPUT);
  
  digitalWrite(BUZZER, LOW); 
  
  
  Serial.println("Iniciando Control ...");
}
 
void loop()
{     
    if(BT.available())
    {
      CharIN = BT.read();
      Serial.print(CharIN);
    
      if(CharIN == 'a'){digitalWrite(BUZZER, HIGH);}
      if(CharIN == 'b'){digitalWrite(BUZZER, LOW);}
    
      
    }
}


