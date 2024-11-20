# wsecurity

esta aplicacion es para poder reducir los casos de violencia contra las mujeres y poder brindarles una mayor confianza para poder denunciar cualquier abuso o violencia contra ellas.


este es el codigo que se utlizo para crear la aplicacion que se espera ser utilizada por la usuaria:


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

A continuacion este es el codigo de la base de datos que va a ser utilizada para almacenar los datos de la usuaria tales como su nombre, direccion, y a su vez la opcion de mandar apoyo inmediato a la ubicacion de la usaria que levanto el reporte. Esto nos ayuda atener mayyo informacion de la persona y la situacion que esta pasando a si mismo cuenta con una opcion de generar una llamada falsa para brindar el apoyo si la usaria se encuentra en un lugar desolado con esto querermos lograr bajaer la taza de secuestros, asaltos, abusos contra las mujeres 


import Types "Types";
import Cycles "mo:base/ExperimentalCycles";
import Text "mo:base/Text";
import Blob "mo:base/Blob";
import Debug "mo:base/Debug";

actor {
  let ic : Types.IC = actor "aaaaa-aa"; // Management Canister ID
  let ubicacion = "l";
  let nombre_de_la_persona = "q";
  let emergencia = "r";

  func fetchSensorData() : async Text {
    // Definir URL
    let url: Text = "https://woman-security.onrender.com/api/seguridad"; // URL corregido

    // Definir encabezados HTTP
    let request_headers = [
      { name = "Host"; value = "woman-security.onrender.com" }, // Corregir "Host"
      { name = "User-Agent"; value = "Motoko HTTP Agent" },
    ];

    // Preparar la solicitud HTTP
    let http_request : Types.HttpRequestArgs = {
        url = url;
        max_response_bytes = null;
        headers = request_headers;
        body = null;
        method = #get;
        transform = null;
    };

    // Añadir ciclos para pagar la solicitud HTTP
    Cycles.add(20_949_972);

    // Realizar la solicitud HTTP y esperar la respuesta
    let http_response : Types.HttpResponsePayload = await ic.http_request(http_request);

    // Decodificar el cuerpo de la respuesta
    let response_body: Blob = Blob.fromArray(http_response.body);
    let decoded_text: Text = switch (Text.decodeUtf8(response_body)) {
      case (null) { "No value returned" };
      case (?text) { text };
    };

    // Retornar el texto decodificado
    return decoded_text;
  };

  public func sendSensorData(sensorData: Types.SensorData) : async Text {
    // Codificar los datos del sensor en formato JSON
    let sensorJson: Text = encodeSensorJson(sensorData);
    Debug.print("Sending JSON: " # sensorJson);

    // URL a la que se enviarán los datos
    let url: Text = "https://wsecurity.com/api/seguridad";

    // Codificar el cuerpo de la solicitud en bytes
    let bodyBytesBlob = Text.encodeUtf8(sensorJson);
    let bodyBytes = Blob.toArray(bodyBytesBlob); // Convertir Blob a [Nat8]

    // Preparar los encabezados y el cuerpo de la solicitud HTTP
    let request_headers = [
      { name = "Host"; value = "wsecurity.com" },
      { name = "Content-Type"; value = "application/json" }
    ];
    let http_request : Types.HttpRequestArgs = {
        url = url;
        max_response_bytes = null;
        headers = request_headers;
        body = ?bodyBytes;
        method = #post;
        transform = null;
    };

    // Añadir ciclos para pagar la solicitud HTTP
    Cycles.add(1_000_000_000_000); // La cantidad específica necesitará ser ajustada según la documentación

    // Realizar la solicitud HTTP POST y esperar la respuesta
    let http_response : Types.HttpResponsePayload = await ic.http_request(http_request);

    // Decodificar y retornar la respuesta
    let response_body: Blob = Blob.fromArray(http_response.body);
    let decoded_text: Text = switch (Text.decodeUtf8(response_body)) {
      case (null) { "No value returned" };
      case (?text) { text };
    };

    return decoded_text;
  };

  private func encodeSensorJson(sensorData: Types.SensorData) : Text{
    // Convertir valores a texto antes de concatenar
    let ubicacionText = "\"" # ubicacion # "\"";
    let nombre_de_la_personaTextex = "\"" # nombre_de_la_persona # "\"";
    let emergenciaTextex = "\"" # emergencia # "\"";
    let comparisonResult = Text.compare(ubicacionText, nombre_de_la_personaTextex);

    // Construyendo la cadena JSON manualmente con claves en minúsculas
    "{ \"ubicacion\": " # ubicacionText # ", \"nombre_de_la_persona\": " # nombre_de_la_personaTextex # ", \"emergencia\": " # emergenciaText # " }"
  }
}
 Finalmente despues de juntar todos estos codigos nos da la seguridad que queremos darles a nuestras usuarias para que reporten cualquier acto de violencia hacia ellas.

Este proyecto tiene como funcionalidiad que cada mujer tènga con ella un boton de panico de bolsiilo como los que se encuentran ubicados en la ciudad de aguascalientes  y con esto nos brinda una atencion mas rapida y efevtiva 

