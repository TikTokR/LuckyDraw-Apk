# LuckyDraw-Apk
I'll include a detailed README.md file in the GitHub repository with:  ✅ Project Overview (What the app does) ✅ Features (User registration, investment, payment verification, winner selection, etc.) ✅ Installation Guide (How to set up the project in Android Studio) ✅ Usage Instructions (How users can invest and check results) ✅ Admin Guide



package com.TikTok R.luckydraw;

import android.app.AlertDialog;
import android.content.DialogInterface;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import java.util.ArrayList;
import java.util.Random;

public class MainActivity extends AppCompatActivity {

    private EditText etName, etPhone, etAmount, etScreenshot;
    private Button btnInvest, btnShowWinner;
    private TextView tvTotalAmount, tvWinner;
    
    private SharedPreferences sharedPreferences;
    private ArrayList<String> users = new ArrayList<>();
    private ArrayList<Integer> amounts = new ArrayList<>();
    
    private static final String PREFS_NAME = "LuckyDrawPrefs";
    private static final String TOTAL_AMOUNT_KEY = "TotalAmount";
    private static final int DRAW_INTERVAL_DAYS = 15; // Lucky Draw every 15 days

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etPhone = findViewById(R.id.etPhone);
        etAmount = findViewById(R.id.etAmount);
        etScreenshot = findViewById(R.id.etScreenshot);
        btnInvest = findViewById(R.id.btnInvest);
        btnShowWinner = findViewById(R.id.btnShowWinner);
        tvTotalAmount = findViewById(R.id.tvTotalAmount);
        tvWinner = findViewById(R.id.tvWinner);

        sharedPreferences = getSharedPreferences(PREFS_NAME, MODE_PRIVATE);
        updateTotalAmountDisplay();

        btnInvest.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                investAmount();
            }
        });

        btnShowWinner.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                selectWinner();
            }
        });
    }

    private void investAmount() {
        String name = etName.getText().toString();
        String phone = etPhone.getText().toString();
        String amountStr = etAmount.getText().toString();
        String screenshot = etScreenshot.getText().toString(); // Manual verification

        if (name.isEmpty() || phone.isEmpty() || amountStr.isEmpty() || screenshot.isEmpty()) {
            Toast.makeText(this, "Please fill all fields!", Toast.LENGTH_SHORT).show();
            return;
        }

        int amount = Integer.parseInt(amountStr);
        if (amount < 1 || amount > 10) {
            Toast.makeText(this, "Investment must be between Rs.1 and Rs.10!", Toast.LENGTH_SHORT).show();
            return;
        }

        // Add user to the list
        users.add(name + " (" + phone + ")");
        amounts.add(amount);

        // Update total amount
        int totalAmount = sharedPreferences.getInt(TOTAL_AMOUNT_KEY, 0) + amount;
        sharedPreferences.edit().putInt(TOTAL_AMOUNT_KEY, totalAmount).apply();
        updateTotalAmountDisplay();

        Toast.makeText(this, "Investment Submitted! Awaiting Verification.", Toast.LENGTH_LONG).show();
    }

    private void updateTotalAmountDisplay() {
        int totalAmount = sharedPreferences.getInt(TOTAL_AMOUNT_KEY, 0);
        tvTotalAmount.setText("Total Invested Amount: Rs. " + totalAmount);
    }

    private void selectWinner() {
        if (users.isEmpty()) {
            Toast.makeText(this, "No participants yet!", Toast.LENGTH_SHORT).show();
            return;
        }

        // Select a random winner
        Random random = new Random();
        int winnerIndex = random.nextInt(users.size());
        String winner = users.get(winnerIndex);

        // Calculate prize distribution
        int totalAmount = sharedPreferences.getInt(TOTAL_AMOUNT_KEY, 0);
        int winnerPrize = (int) (totalAmount * 0.65); // 65% to winner
        int adminShare = (int) (totalAmount * 0.35); // 35% to admin

        // Show winner dialog
        new AlertDialog.Builder(this)
                .setTitle("Lucky Draw Winner")
                .setMessage("Winner: " + winner + "\nPrize: Rs. " + winnerPrize + "\nAdmin Share: Rs. " + adminShare)
                .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        resetLuckyDraw();
                    }
                })
                
res/layouts/main_activity.xml


<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="20dp"
    android:orientation="vertical">

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Name" />

    <EditText
        android:id="@+id/etPhone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Phone Number" />

    <EditText
        android:id="@+id/etAmount"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Investment Amount (Rs.1 - Rs.10)"
        android:inputType="number" />

    <EditText
        android:id="@+id/etScreenshot"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Screenshot Link (Manual Verification)" />

    <Button
        android:id="@+id/btnInvest"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Invest Now" />

    <TextView
        android:id="@+id/tvTotalAmount"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Total Invested Amount: Rs. 0"
        android:textSize="18sp"
        android:padding="10dp" />

    <Button
        android:id="@+id/btnShowWinner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Show Lucky Draw Winner" />

    <TextView
        android:id="@+id/tvWinner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Winner: TBD"
        android:textSize="18sp"
        android:padding="10dp" />

</LinearLayout>
        tvWinner.setText("Winner: " + winner + " (Prize: Rs. " + winnerPrize + ")");
    }

    private void resetLuckyDraw() {
        sharedPreferences.edit().putInt(TOTAL_AMOUNT_KEY, 0).apply();
        users.clear();
        amounts.clear();
        updateTotalAmountDisplay();
        tvWinner.setText("Winner: TBD");
        Toast.makeText(this, "New Lucky Draw Started!", Toast.LENGTH_SHORT).show();
    }
}

