// UPDATED Account.java (Login Activity)
package com.example.shareitapp;

import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.SetOptions;

import java.util.HashMap;
import java.util.Map;

public class Account extends AppCompatActivity implements View.OnClickListener {
    Button btnSubmit;
    EditText etEmail, etPassword;
    FirebaseAuth auth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_account);

        btnSubmit = findViewById(R.id.btnSubmitLogin);
        etEmail = findViewById(R.id.etEmail);
        etPassword = findViewById(R.id.etPassword);
        btnSubmit.setOnClickListener(this);
        auth = FirebaseAuth.getInstance();
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (auth.getCurrentUser() != null) {
            startActivity(new Intent(Account.this, MainActivity.class));
            finish();
        }
    }

    // ADD THIS METHOD
    private void saveUserToFirestore() {
        FirebaseUser user = FirebaseAuth.getInstance().getCurrentUser();
        if (user != null) {
            Map<String, Object> userData = new HashMap<>();
            userData.put("uid", user.getUid());
            userData.put("displayName", user.getDisplayName());
            userData.put("email", user.getEmail());

            FirebaseFirestore.getInstance()
                    .collection("users")
                    .document(user.getUid()) // Use UID as document ID
                    .set(userData, SetOptions.merge()) // Use merge to avoid overwriting existing data
                    .addOnSuccessListener(aVoid -> Log.d("Firestore", "User data saved successfully"))
                    .addOnFailureListener(e -> Log.e("Firestore", "Error saving user data", e));
        }
    }

    @Override
    public void onClick(View v) {
        if (v == btnSubmit) {
            String mail = etEmail.getText().toString();
            String pass = etPassword.getText().toString();

            if (mail.isEmpty() || pass.isEmpty()) {
                Toast.makeText(this, "Fill all the fields", Toast.LENGTH_LONG).show();
            } else {
                auth.signInWithEmailAndPassword(mail, pass)
                        .addOnCompleteListener(this, task -> {
                            if (task.isSuccessful()) {
                                Toast.makeText(Account.this, "Login Successful", Toast.LENGTH_SHORT).show();

                                // ADD THIS LINE - Save user data to Firestore after successful login
                                saveUserToFirestore();

                                startActivity(new Intent(Account.this, MainActivity.class));
                                finish();
                            } else {
                                Toast.makeText(Account.this, "Login Failed", Toast.LENGTH_SHORT).show();
                            }
                        });
            }
        }
    }
}

package com.example.shareitapp;

public class Assignment {
    private String id;
    private String NameOfSubject;
    private String subTopic;
    private int daySub;
    private int monthSub;
    private int yearSub;
    private String uploaderId;


    public Assignment() {}

    public String getNameOfSubject() {
        return NameOfSubject;
    }

    public void setNameOfSubject(String nameOfSubject) {
        NameOfSubject = nameOfSubject;
    }

    public String getSubTopic() {
        return subTopic;
    }

    public void setSubTopic(String subTopic) {
        this.subTopic = subTopic;
    }

    public int getDaySub() {
        return daySub;
    }

    public void setDaySub(int daySub) {
        this.daySub = daySub;
    }

    public int getMonthSub() {
        return monthSub;
    }

    public void setMonthSub(int monthSub) {
        this.monthSub = monthSub;
    }

    public int getYearSub() {
        return yearSub;
    }

    public void setYearSub(int yearSub) {
        this.yearSub = yearSub;
    }

    public String getUploaderId() {
        return uploaderId;
    }

    public void setUploaderId(String uploaderId) {
        this.uploaderId = uploaderId;
    }

    public String getId(){
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Assignment(String nameOfSubject, String subTopic, int daySub, int monthSub, int yearSub, String uploaderId) {
        this.NameOfSubject = nameOfSubject;
        this.subTopic = subTopic;
        this.daySub = daySub;
        this.monthSub = monthSub;
        this.yearSub = yearSub;
        this.uploaderId = uploaderId;
    }

}

package com.example.shareitapp;

import android.content.Context;
import android.util.Log;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.firestore.FirebaseFirestore;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Locale;
import java.util.Map;

public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.MessageViewHolder> {
    private Context context;
    private List<Message> messageList;
    private String currentUserId;
    private FirebaseFirestore db;
    private Map<String, String> userNameCache; // Cache usernames to avoid repeated queries

    public ChatAdapter(Context context, List<Message> messageList) {
        this.context = context;
        this.messageList = messageList;
        this.currentUserId = FirebaseAuth.getInstance().getCurrentUser().getUid();
        this.db = FirebaseFirestore.getInstance();
        this.userNameCache = new HashMap<>();
    }

    @NonNull
    @Override
    public MessageViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.message_item, parent, false);
        return new MessageViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull MessageViewHolder holder, int position) {
        Message message = messageList.get(position);
        holder.messageText.setText(message.getMessageText());

        // Format timestamp
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm", Locale.getDefault());
        String time = sdf.format(new Date(message.getTimestamp()));
        holder.timeText.setText(time);

        // Check if message is from current user
        boolean isCurrentUser = message.getSenderId().equals(currentUserId);

        // Set username
        String senderId = message.getSenderId();
        if (isCurrentUser) {
            holder.senderNameText.setText("You");
            holder.senderNameText.setVisibility(View.GONE); // Hide "You" for sent messages
        } else {
            holder.senderNameText.setVisibility(View.VISIBLE);
            // Check cache first
            if (userNameCache.containsKey(senderId)) {
                holder.senderNameText.setText(userNameCache.get(senderId));
            } else {
                holder.senderNameText.setText("Loading...");
                // Fetch username from Firestore - CHANGED: Look for 'displayName' instead of 'username'
                db.collection("users").document(senderId).get()
                        .addOnSuccessListener(document -> {
                            if (document.exists()) {
                                // Try displayName first, then fall back to email if displayName is null
                                String displayName = document.getString("displayName");
                                String email = document.getString("email");

                                String username;
                                if (displayName != null && !displayName.isEmpty()) {
                                    username = displayName;
                                } else if (email != null && !email.isEmpty()) {
                                    username = email; // Fallback to email
                                } else {
                                    username = "Unknown User";
                                }

                                userNameCache.put(senderId, username); // Cache it
                                holder.senderNameText.setText(username);
                                Log.d("ChatAdapter", "Username loaded: " + username);
                            } else {
                                holder.senderNameText.setText("Unknown User");
                                Log.w("ChatAdapter", "User document doesn't exist for: " + senderId);
                            }
                        })
                        .addOnFailureListener(e -> {
                            Log.e("ChatAdapter", "Failed to load username", e);
                            holder.senderNameText.setText("Unknown User");
                        });
            }
        }

        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
        );

        if (isCurrentUser) {
            // Sent message - align to right with blue background
            params.gravity = Gravity.END;
            holder.messageContainer.setGravity(Gravity.END);
            holder.messageText.setBackgroundResource(R.drawable.bg_message_sent);
            holder.messageText.setTextColor(context.getResources().getColor(android.R.color.black));
            holder.timeText.setGravity(Gravity.END);
        } else {
            // Received message - align to left with gray background
            params.gravity = Gravity.START;
            holder.messageContainer.setGravity(Gravity.START);
            holder.messageText.setBackgroundResource(R.drawable.bg_message_received);
            holder.messageText.setTextColor(context.getResources().getColor(android.R.color.black));
            holder.timeText.setGravity(Gravity.START);
        }

        holder.messageContainer.setLayoutParams(params);
    }

    @Override
    public int getItemCount() {
        return messageList.size();
    }

    public static class MessageViewHolder extends RecyclerView.ViewHolder {
        TextView messageText, timeText, senderNameText;
        LinearLayout messageContainer;

        public MessageViewHolder(@NonNull View itemView) {
            super(itemView);
            messageText = itemView.findViewById(R.id.messageText);
            timeText = itemView.findViewById(R.id.timeText);
            senderNameText = itemView.findViewById(R.id.senderNameText);
            messageContainer = itemView.findViewById(R.id.messageContainer);
        }
    }
}

package com.example.shareitapp;

import android.os.Bundle;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.example.shareitapp.Message;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;

import com.google.firebase.auth.FirebaseAuth;import com.google.firebase.firestore.DocumentSnapshot;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.Query;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class ChatFragment extends Fragment {

    private RecyclerView chatRecyclerView;
    private EditText messageEditText;
    private ImageButton sendButton;
    private ChatAdapter chatAdapter;
    private List<Message> messageList;
    private FirebaseFirestore db;
    private String senderId, receiverId, chatRoomId, assignmentId;

    public ChatFragment() {
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);

        chatRecyclerView = view.findViewById(R.id.chatRecyclerView);
        messageEditText = view.findViewById(R.id.messageEditText);
        sendButton = view.findViewById(R.id.sendButton);
        db = FirebaseFirestore.getInstance();

        senderId = FirebaseAuth.getInstance().getCurrentUser().getUid();
        if (getArguments() != null) {
            receiverId = getArguments().getString("receiverId");
            assignmentId = getArguments().getString("assignmentId");
        }
        chatRoomId = generateChatRoomId(senderId, receiverId, assignmentId);

        messageList = new ArrayList<>();
        chatAdapter = new ChatAdapter(getContext(), messageList);
        chatRecyclerView.setAdapter(chatAdapter);
        chatRecyclerView.setLayoutManager(new LinearLayoutManager(getContext()));

        loadMessages();

        sendButton.setOnClickListener(v -> sendMessage());

        return view;
    }

    private void loadMessages() {
        db.collection("chats").document(chatRoomId).collection("messages")
                .orderBy("timestamp", Query.Direction.ASCENDING)
                .addSnapshotListener((value, error) -> {
                    if (error != null) return;
                    messageList.clear();
                    for (DocumentSnapshot doc : value.getDocuments()) {
                        messageList.add(doc.toObject(Message.class));
                    }
                    chatAdapter.notifyDataSetChanged();
                    chatRecyclerView.scrollToPosition(messageList.size() - 1);
                });
    }

    private void sendMessage() {
        String message = messageEditText.getText().toString().trim();
        if (!message.isEmpty()) {
            Message msg = new Message(senderId, receiverId, message, System.currentTimeMillis());

            // Save the message
            db.collection("chats").document(chatRoomId).collection("messages").add(msg);

            // Ensure the parent chat document exists
            db.collection("chats").document(chatRoomId).set(new HashMap<>());

            saveChatMetadata(senderId, receiverId, chatRoomId, assignmentId, message);


            messageEditText.setText("");
        }
    }


    private void saveChatMetadata(String senderId, String receiverId, String chatRoomId, String assignmentId, String lastMessage) {
        long timestamp = System.currentTimeMillis();
        HashMap<String, Object> data = new HashMap<>();
        data.put("chatRoomId", chatRoomId);
        data.put("otherUserId", receiverId);
        data.put("assignmentId", assignmentId);
        data.put("lastMessage", lastMessage);
        data.put("timestamp", timestamp);

        // Save under sender
        db.collection("userChats").document(senderId)
                .collection("chats").document(chatRoomId).set(data);

        // Save under receiver (flip sender/receiver)
        data.put("otherUserId", senderId);
        db.collection("userChats").document(receiverId)
                .collection("chats").document(chatRoomId).set(data);
    }

    private String generateChatRoomId(String senderId, String receiverId, String assignmentId) {
        String sortedUsers = senderId.compareTo(receiverId) < 0
                ? senderId + "_" + receiverId
                : receiverId + "_" + senderId;
        return sortedUsers + "_" + assignmentId;
    }
}




package com.example.shareitapp;

import android.os.Bundle;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.example.shareitapp.Message;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;

import com.google.firebase.auth.FirebaseAuth;import com.google.firebase.firestore.DocumentSnapshot;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.Query;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class ChatFragment extends Fragment {

    private RecyclerView chatRecyclerView;
    private EditText messageEditText;
    private ImageButton sendButton;
    private ChatAdapter chatAdapter;
    private List<Message> messageList;
    private FirebaseFirestore db;
    private String senderId, receiverId, chatRoomId, assignmentId;

    public ChatFragment() {
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);

        chatRecyclerView = view.findViewById(R.id.chatRecyclerView);
        messageEditText = view.findViewById(R.id.messageEditText);
        sendButton = view.findViewById(R.id.sendButton);
        db = FirebaseFirestore.getInstance();

        senderId = FirebaseAuth.getInstance().getCurrentUser().getUid();
        if (getArguments() != null) {
            receiverId = getArguments().getString("receiverId");
            assignmentId = getArguments().getString("assignmentId");
        }
        chatRoomId = generateChatRoomId(senderId, receiverId, assignmentId);

        messageList = new ArrayList<>();
        chatAdapter = new ChatAdapter(getContext(), messageList);
        chatRecyclerView.setAdapter(chatAdapter);
        chatRecyclerView.setLayoutManager(new LinearLayoutManager(getContext()));

        loadMessages();

        sendButton.setOnClickListener(v -> sendMessage());

        return view;
    }

    private void loadMessages() {
        db.collection("chats").document(chatRoomId).collection("messages")
                .orderBy("timestamp", Query.Direction.ASCENDING)
                .addSnapshotListener((value, error) -> {
                    if (error != null) return;
                    messageList.clear();
                    for (DocumentSnapshot doc : value.getDocuments()) {
                        messageList.add(doc.toObject(Message.class));
                    }
                    chatAdapter.notifyDataSetChanged();
                    chatRecyclerView.scrollToPosition(messageList.size() - 1);
                });
    }

    private void sendMessage() {
        String message = messageEditText.getText().toString().trim();
        if (!message.isEmpty()) {
            Message msg = new Message(senderId, receiverId, message, System.currentTimeMillis());

            // Save the message
            db.collection("chats").document(chatRoomId).collection("messages").add(msg);

            // Ensure the parent chat document exists
            db.collection("chats").document(chatRoomId).set(new HashMap<>());

            saveChatMetadata(senderId, receiverId, chatRoomId, assignmentId, message);


            messageEditText.setText("");
        }
    }


    private void saveChatMetadata(String senderId, String receiverId, String chatRoomId, String assignmentId, String lastMessage) {
        long timestamp = System.currentTimeMillis();
        HashMap<String, Object> data = new HashMap<>();
        data.put("chatRoomId", chatRoomId);
        data.put("otherUserId", receiverId);
        data.put("assignmentId", assignmentId);
        data.put("lastMessage", lastMessage);
        data.put("timestamp", timestamp);

        // Save under sender
        db.collection("userChats").document(senderId)
                .collection("chats").document(chatRoomId).set(data);

        // Save under receiver (flip sender/receiver)
        data.put("otherUserId", senderId);
        db.collection("userChats").document(receiverId)
                .collection("chats").document(chatRoomId).set(data);
    }

    private String generateChatRoomId(String senderId, String receiverId, String assignmentId) {
        String sortedUsers = senderId.compareTo(receiverId) < 0
                ? senderId + "_" + receiverId
                : receiverId + "_" + senderId;
        return sortedUsers + "_" + assignmentId;
    }
}



package com.example.shareitapp;

import android.content.Context;
import android.os.Bundle;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.firestore.FirebaseFirestore;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;

public class ChatListAdapter extends RecyclerView.Adapter<ChatListAdapter.ViewHolder>{
    private Context context;
    private List <ChatInfo> chatInfos;

    public ChatListAdapter(Context context, List<ChatInfo> chatInfos) {
        this.context = context;
        this.chatInfos = chatInfos;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.chat_item, parent, false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        ChatInfo chatInfo = chatInfos.get(position);
        String userId = chatInfo.userId;
        String assignmentId = chatInfo.assignmentId;

        Log.d("ChatListAdapter", "Binding chat item for user: " + userId);

        // Set default values first
        holder.usernameText.setText("Loading...");
        holder.lastMessageText.setText(chatInfo.lastMessage != null ? chatInfo.lastMessage : "No messages yet");

        // Format and set timestamp
        if (chatInfo.timestamp > 0) {
            SimpleDateFormat sdf = new SimpleDateFormat("MMM dd, HH:mm", Locale.getDefault());
            String timeStr = sdf.format(new Date(chatInfo.timestamp));
            holder.timeText.setText(timeStr);
        } else {
            holder.timeText.setText("");
        }

        // Fetch and display username - CHANGED: Look for 'displayName' instead of 'username'
        FirebaseFirestore.getInstance().collection("users").document(userId).get()
                .addOnSuccessListener(document -> {
                    if (document.exists()) {
                        // Try displayName first, then fall back to email if displayName is null
                        String displayName = document.getString("displayName");
                        String email = document.getString("email");

                        String username;
                        if (displayName != null && !displayName.isEmpty()) {
                            username = displayName;
                        } else if (email != null && !email.isEmpty()) {
                            username = email; // Fallback to email
                        } else {
                            username = "Unknown User";
                        }

                        holder.usernameText.setText(username);
                        Log.d("ChatListAdapter", "Username loaded: " + username);
                    } else {
                        holder.usernameText.setText("Unknown User");
                        Log.w("ChatListAdapter", "User document doesn't exist for: " + userId);
                    }
                })
                .addOnFailureListener(e -> {
                    holder.usernameText.setText("Unknown User");
                    Log.e("ChatListAdapter", "Failed to load username", e);
                });

        // Set click listener
        holder.itemView.setOnClickListener(v -> {
            Log.d("ChatListAdapter", "Chat item clicked - Opening chat with: " + userId);

            ChatFragment chatFragment = new ChatFragment();
            Bundle bundle = new Bundle();
            bundle.putString("receiverId", userId);
            bundle.putString("assignmentId", assignmentId);
            chatFragment.setArguments(bundle);

            ((AppCompatActivity) context).getSupportFragmentManager().beginTransaction()
                    .replace(R.id.fl, chatFragment)
                    .addToBackStack(null)
                    .commit();
        });
    }

    @Override
    public int getItemCount() {
        Log.d("ChatListAdapter", "Total chat items: " + chatInfos.size());
        return chatInfos.size();
    }

    public static class ViewHolder extends RecyclerView.ViewHolder {
        TextView usernameText, lastMessageText, timeText;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            usernameText = itemView.findViewById(R.id.chatUserNameTextView);
            lastMessageText = itemView.findViewById(R.id.lastMessageTextView);
            timeText = itemView.findViewById(R.id.timeTextView);
        }
    }
}


package com.example.shareitapp;

import android.app.AlertDialog;
import android.os.Bundle;
import android.text.InputType;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.LinearLayout;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.example.shareitapp.VideoAdapter;
import com.example.shareitapp.YouTubeVideo;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.QueryDocumentSnapshot;
import java.util.ArrayList;
import java.util.List;

public class ClassFragment extends Fragment {
    private RecyclerView recyclerView;
    private VideoAdapter adapter;
    private List<YouTubeVideo> videoList;
    private FirebaseFirestore db;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_class, container, false);

        db = FirebaseFirestore.getInstance();
        recyclerView = view.findViewById(R.id.recyclerView);
        recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        videoList = new ArrayList<>();
        adapter = new VideoAdapter(getContext(), videoList);
        recyclerView.setAdapter(adapter);

        ImageButton addVideoButton = view.findViewById(R.id.addVideoButton);
        addVideoButton.setOnClickListener(v -> showUploadDialog());

        loadVideos();
        return view;
    }

    private void showUploadDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(getContext());
        builder.setTitle("Upload Video");

        LinearLayout layout = new LinearLayout(getContext());
        layout.setOrientation(LinearLayout.VERTICAL);

        EditText urlInput = new EditText(getContext());
        urlInput.setHint("YouTube URL");
        layout.addView(urlInput);

        EditText subjectInput = new EditText(getContext());
        subjectInput.setHint("Subject Name");
        layout.addView(subjectInput);

        builder.setView(layout);
        builder.setPositiveButton("Upload", (dialog, which) -> {
            String url = urlInput.getText().toString().trim();
            String subject = subjectInput.getText().toString().trim();

            if (!url.isEmpty() && !subject.isEmpty()) {
                db.collection("videos").add(new YouTubeVideo(url, subject));
                loadVideos();
            }
        });
        builder.setNegativeButton("Cancel", (dialog, which) -> dialog.cancel());
        builder.show();
    }

    private void loadVideos() {
        db.collection("videos").get().addOnSuccessListener(query -> {
            videoList.clear();
            for (QueryDocumentSnapshot doc : query) {
                videoList.add(doc.toObject(YouTubeVideo.class));
            }
            adapter.notifyDataSetChanged();
        });
    }
}




package com.example.shareitapp;

import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ProgressBar;

import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.firestore.DocumentChange;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.FirebaseFirestoreException;
import com.google.firebase.firestore.Query;
import com.google.firebase.firestore.QuerySnapshot;

import java.util.ArrayList;
import java.util.HashMap;

public class HomeFragment extends Fragment implements View.OnClickListener {

    RecyclerView recyclerView;
    MyAdapter myAdapter;
    ArrayList<Assignment> assignmentList;
    FirebaseFirestore db;
    ProgressBar progressBar;
    View view;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        view = inflater.inflate(R.layout.fragment_home, container, false);

        progressBar = view.findViewById(R.id.progressBar);
        recyclerView = view.findViewById(R.id.recyclerView);
        recyclerView.setHasFixedSize(true);
        recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));

        db = FirebaseFirestore.getInstance();
        assignmentList = new ArrayList<>();
        myAdapter = new MyAdapter(getContext(), assignmentList, (uploaderId, assignmentId) -> {

            String currentUserId = FirebaseAuth.getInstance().getCurrentUser().getUid();
            String chatId = currentUserId.compareTo(uploaderId) < 0 ?
                    currentUserId + "_" + uploaderId : uploaderId + "_" + currentUserId;

            // Create chat if not exists
            FirebaseFirestore.getInstance().collection("chats").document(chatId).set(new HashMap<>())
                    .addOnSuccessListener(aVoid -> Log.d("Chat", "Chat created or already exists"))
                    .addOnFailureListener(e -> Log.e("Chat", "Failed to create chat", e));
            ChatFragment chatFragment = new ChatFragment();
            Bundle bundle = new Bundle();
            bundle.putString("receiverId", uploaderId);
            bundle.putString("assignmentId", assignmentId);
            bundle.putString("currentUserId", FirebaseAuth.getInstance().getCurrentUser().getUid());
            chatFragment.setArguments(bundle);

            if (getActivity() != null && !getActivity().isFinishing()) {
                getActivity().getSupportFragmentManager().beginTransaction()
                        .replace(R.id.fl, chatFragment)
                        .addToBackStack(null)
                        .commit();
            }
        },null,  false);
        recyclerView.setAdapter(myAdapter);


        progressBar.setVisibility(View.VISIBLE);

        EventChangeListener();


        return view;
    }

    private void EventChangeListener() {
        db.collection("assignments").orderBy("nameOfSubject", Query.Direction.ASCENDING)
                .addSnapshotListener((value, error) -> {

                    if (error != null) {
                        progressBar.setVisibility(View.GONE);
                        Log.e("Firestore Error", error.getMessage());
                        return;
                    }

                    if (value != null) {
                        for (DocumentChange dc : value.getDocumentChanges()) {
                            if (dc.getType() == DocumentChange.Type.ADDED) {
                                Assignment assignment = dc.getDocument().toObject(Assignment.class);
                                assignment.setId(dc.getDocument().getId());  //  Set Firestore doc ID
                                assignmentList.add(assignment);
                            }
                        }

                        myAdapter.notifyDataSetChanged();
                    }

                    progressBar.setVisibility(View.GONE);
                });
    }

    @Override
    public void onClick(View v) {

    }
}




package com.example.shareitapp;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ImageButton;
import android.widget.PopupMenu;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;

import com.google.android.material.bottomnavigation.BottomNavigationView;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.firebase.Firebase;
import com.google.firebase.auth.FirebaseAuth;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    BottomNavigationView bottomNavigationView;
    ImageButton btnMenu;
    FirebaseAuth auth;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bottomNavigationView = findViewById(R.id.bnv);
        btnMenu = findViewById(R.id.btnMenu);
        auth = FirebaseAuth.getInstance();

        FloatingActionButton messageButton = findViewById(R.id.messageButton);
        messageButton.setOnClickListener(v -> {
            MessagesFragment messagesFragment = new MessagesFragment();
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.fl, messagesFragment)
                    .addToBackStack(null)
                    .commit();
        });

        btnMenu.setOnClickListener(view -> {
            PopupMenu popupMenu = new PopupMenu(MainActivity.this, view);
            popupMenu.getMenuInflater().inflate(R.menu.menu_popup, popupMenu.getMenu());

            popupMenu.setOnMenuItemClickListener(item -> {
                if (item.getItemId() == R.id.action_logout) {
                    FirebaseAuth.getInstance().signOut();
                    Intent intent = new Intent(MainActivity.this, Register.class);
                    startActivity(intent);
                    finish();
                    return true;
                }
                return false;
            });

            popupMenu.show();
        });


        bottomNavigationView.setOnItemReselectedListener(item -> {
            if (item.getItemId() == R.id.home)
                replaceFragment(new HomeFragment());
            else if (item.getItemId() == R.id.profile)
                replaceFragment(new ProfileFragment());
            else if (item.getItemId() == R.id.Class)
                replaceFragment(new ClassFragment());
//            else if (item.getItemId() == R.id)
//                replaceFragment(new Quiz());
        });

    }

    public void replaceFragment(Fragment fragment) {
        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        fragmentTransaction.replace(R.id.fl, fragment);
        fragmentTransaction.commit();
    }

    @Override
    public void onClick(View v) {
    }
}




package com.example.shareitapp;

public class Message {
    private String senderId;
    private String receiverId;
    private String messageText;
    private long timestamp;


    public Message() {}

    public Message(String senderId, String receiverId, String messageText, long timestamp) {
        this.senderId = senderId;
        this.receiverId = receiverId;
        this.messageText = messageText;
        this.timestamp = timestamp;
    }

    public String getSenderId() {
        return senderId;
    }

    public String getReceiverId() {
        return receiverId;
    }

    public String getMessageText() {
        return messageText;
    }

    public long getTimestamp() {
        return timestamp;
    }
}


package com.example.shareitapp;

import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.firestore.DocumentSnapshot;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.Query;

import java.util.ArrayList;
import java.util.List;

public class MessagesFragment extends Fragment {

    private RecyclerView chatsRecyclerView;
    private FirebaseFirestore db;
    private String currentUserId;
    private List<ChatInfo> chatInfos;
    private ChatListAdapter adapter;
    private TextView noChatsTextView;

    public MessagesFragment() {

    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_messages, container, false);


        chatsRecyclerView = view.findViewById(R.id.chatRecyclerView);
        noChatsTextView = view.findViewById(R.id.noChatsTextView);


        chatsRecyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        chatsRecyclerView.setHasFixedSize(true); // Performance optimization

        chatInfos = new ArrayList<>();
        adapter = new ChatListAdapter(getContext(), chatInfos);
        chatsRecyclerView.setAdapter(adapter);

        currentUserId = FirebaseAuth.getInstance().getCurrentUser().getUid();
        db = FirebaseFirestore.getInstance();

        Log.d("MessagesFragment", "Fragment created");
        Log.d("MessagesFragment", "Current User ID: " + currentUserId);
        Log.d("MessagesFragment", "RecyclerView setup complete");


        loadChats();

        return view;
    }

    @Override
    public void onResume() {
        super.onResume();
        Log.d("MessagesFragment", "Fragment resumed - refreshing chats");
        if (adapter != null) {
            adapter.notifyDataSetChanged();
        }
    }

    private void loadChats() {
        Log.d("MessagesFragment", "Loading chats for user: " + currentUserId);

        showNoChatsMessage("Loading chats...");

        db.collection("userChats")
                .document(currentUserId)
                .collection("chats")
                .orderBy("timestamp", Query.Direction.DESCENDING)
                .addSnapshotListener((value, error) -> {
                    if (error != null) {
                        Log.e("MessagesFragment", "Error loading chats: ", error);
                        showNoChatsMessage("Error loading chats");
                        return;
                    }

                    if (value == null || value.isEmpty()) {
                        Log.d("MessagesFragment", "No chat data found");
                        showNoChatsMessage("No chats yet");
                        return;
                    }

                    Log.d("MessagesFragment", "Found " + value.size() + " chat documents");

                    chatInfos.clear();
                    for (DocumentSnapshot doc : value.getDocuments()) {
                        String otherUserId = doc.getString("otherUserId");
                        String assignmentId = doc.getString("assignmentId");
                        String chatRoomId = doc.getString("chatRoomId");
                        String lastMessage = doc.getString("lastMessage");
                        Long timestamp = doc.getLong("timestamp");

                        Log.d("MessagesFragment", "Processing chat - Other User: " + otherUserId +
                                ", Assignment: " + assignmentId + ", ChatRoom: " + chatRoomId);

                        if (otherUserId != null && assignmentId != null && chatRoomId != null) {
                            long ts = timestamp != null ? timestamp : 0;
                            chatInfos.add(new ChatInfo(otherUserId, assignmentId, chatRoomId, lastMessage, ts));
                        }
                    }

                    Log.d("MessagesFragment", "Total chats loaded: " + chatInfos.size());

                    // Update UI on main thread
                    new Handler(Looper.getMainLooper()).post(() -> {
                        if (chatInfos.isEmpty()) {
                            showNoChatsMessage("No chats yet");
                        } else {
                            hideNoChatsMessage();
                            adapter.notifyDataSetChanged();
                        }
                    });
                });
    }

    private void showNoChatsMessage(String message) {
        if (noChatsTextView != null) {
            noChatsTextView.setText(message);
            noChatsTextView.setVisibility(View.VISIBLE);
            chatsRecyclerView.setVisibility(View.GONE);
        }
    }

    private void hideNoChatsMessage() {
        if (noChatsTextView != null) {
            noChatsTextView.setVisibility(View.GONE);
            chatsRecyclerView.setVisibility(View.VISIBLE);
        }
    }
}




package com.example.shareitapp;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.content.Context;
import android.widget.Button;
import android.widget.CalendarView;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.FragmentTransaction;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.firestore.FirebaseFirestore;

import java.util.ArrayList;
import java.util.Calendar;

public class MyAdapter  extends RecyclerView.Adapter<MyAdapter.MyViewHolder>{

    Context context;
    ArrayList<Assignment> list;
    OnRespondClickListener respondClickListener;
    OnDeleteClickListener deleteClickListener;
    boolean isProfileView;


    public interface OnRespondClickListener {
        void onRespondClick(String uploaderId, String assignmentId);
    }

    public interface OnDeleteClickListener {
        void onDeleteClick(String assignmentId);
    }

    public MyAdapter(Context context, ArrayList<Assignment> list, OnRespondClickListener respondClickListener,  OnDeleteClickListener deleteClickListener, boolean isProfileView) {
        this.context = context;
        this.list = list;
        this.respondClickListener = respondClickListener;
        this.deleteClickListener = deleteClickListener;
        this.isProfileView = isProfileView;
    }

    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(context).inflate(R.layout.recycleritem_task,parent,false);
        return new MyViewHolder(v);
    }


    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, int position) {

        Assignment assignment = list.get(position);
        holder.namOfSub.setText(assignment.getNameOfSubject());
        holder.subtopic.setText(assignment.getSubTopic());


        if (assignment.getNameOfSubject().equals("Math")) {
            holder.imageView.setImageResource(R.drawable.math);

        } else if (assignment.getNameOfSubject().equals("English")) {
            holder.imageView.setImageResource(R.drawable.english);

        } else if (assignment.getNameOfSubject().equals("Science")) {
            holder.imageView.setImageResource(R.drawable.science);

        } else if (assignment.getNameOfSubject().equals("History")) {
            holder.imageView.setImageResource(R.drawable.history);

        } else if (assignment.getNameOfSubject().equals("Geography")) {
            holder.imageView.setImageResource(R.drawable.geography);

            } else if (assignment.getNameOfSubject().equals("Art")) {
            holder.imageView.setImageResource(R.drawable.art);

            } else if (assignment.getNameOfSubject().equals("Music")) {
            holder.imageView.setImageResource(R.drawable.music);

            } else if (assignment.getNameOfSubject().equals("Computer Science")) {
            holder.imageView.setImageResource(R.drawable.csmajor);

            } else if (assignment.getNameOfSubject().equals("Physical Education")) {
            holder.imageView.setImageResource(R.drawable.physical_education);

            } else if (assignment.getNameOfSubject().equals("Theater")) {
            holder.imageView.setImageResource(R.drawable.theater);

            } else if (assignment.getNameOfSubject().equals("Sport")) {
            holder.imageView.setImageResource(R.drawable.physical_education);

            } else if (assignment.getNameOfSubject().equals("Literature")) {
            holder.imageView.setImageResource(R.drawable.literature);

            } else if (assignment.getNameOfSubject().equals("Bible")) {
            holder.imageView.setImageResource(R.drawable.bible);

            } else if (assignment.getNameOfSubject().equals("Finance")) {
            holder.imageView.setImageResource(R.drawable.finance);

            } else if (assignment.getNameOfSubject().equals("Biology")) {
            holder.imageView.setImageResource(R.drawable.biology);

            } else if (assignment.getNameOfSubject().equals("Chemistry")) {
            holder.imageView.setImageResource(R.drawable.chemistry);

            } else if (assignment.getNameOfSubject().equals("Physics")) {
            holder.imageView.setImageResource(R.drawable.physics);
        }

        int day = assignment.getDaySub();
        int month = assignment.getMonthSub() + 1;
        int year = assignment.getYearSub();

        String formattedDate = day + "/" + month + "/" + year;
        holder.date.setText(formattedDate);

        holder.respondBtn.setOnClickListener(v -> {
            respondClickListener.onRespondClick(assignment.getUploaderId(), assignment.getId());
        });

        if (isProfileView) {
            holder.deleteIV.setVisibility(View.VISIBLE);
            holder.respondBtn.setVisibility(View.GONE);
            holder.deleteIV.setOnClickListener(v -> {
                deleteClickListener.onDeleteClick(assignment.getId());
            });
        } else {
            holder.deleteIV.setVisibility(View.GONE);
        }

    }


    @Override
    public int getItemCount() {
        return list.size();
    }


    public static class MyViewHolder extends RecyclerView.ViewHolder {


        Button respondBtn;
        TextView namOfSub, subtopic, date;
        ImageView imageView, deleteIV;
        public MyViewHolder(@NonNull View itemView) {
            super(itemView);
            date = itemView.findViewById(R.id.dateTV);
            respondBtn = itemView.findViewById(R.id.respondBtn);
            namOfSub = itemView.findViewById(R.id.nameOfSubTV);
            subtopic = itemView.findViewById(R.id.subTopicTV);
            imageView = itemView.findViewById(R.id.imageIv);
            deleteIV = itemView.findViewById(R.id.deleteIV); // ADD THIS
        }
    }

}



package com.example.shareitapp;

import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.CalendarView;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.Spinner;
import android.widget.TextView;

import androidx.fragment.app.Fragment;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.firestore.QueryDocumentSnapshot;
import com.orhanobut.dialogplus.DialogPlus;
import com.orhanobut.dialogplus.ViewHolder;
import com.google.firebase.firestore.FirebaseFirestore;

import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;
import java.util.UUID;


public class ProfileFragment extends Fragment implements View.OnClickListener{

    RecyclerView profileRecyclerView;
    ArrayList<Assignment> myAssignments;
    MyAdapter myAdapter;
    FirebaseFirestore db;
    FirebaseUser user;
    View view;

    ImageButton ivAddTask;
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        view =inflater.inflate(R.layout.fragment_profile, container, false);
        ivAddTask = view.findViewById(R.id.imageButton);
        ivAddTask.setOnClickListener(this);
        db = FirebaseFirestore.getInstance();
        user = FirebaseAuth.getInstance().getCurrentUser();
        if (user != null) {
            String name = user.getDisplayName();
            TextView nameTextView = view.findViewById(R.id.NameOfUser);
            nameTextView.setText("Hello " + name + "!");
        }
        profileRecyclerView = view.findViewById(R.id.profileRecyclerView);
        profileRecyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        profileRecyclerView.setHasFixedSize(true);

        myAssignments = new ArrayList<>();

        myAdapter = new MyAdapter(getContext(), myAssignments,
                (uploaderId, assignmentId) -> {
                },
                assignmentId -> {
                    FirebaseFirestore.getInstance().collection("assignments").document(assignmentId)
                            .delete()
                            .addOnSuccessListener(aVoid -> {
                                // Remove from local list
                                for (int i = 0; i < myAssignments.size(); i++) {
                                    if (myAssignments.get(i).getId().equals(assignmentId)) {
                                        myAssignments.remove(i);
                                        myAdapter.notifyItemRemoved(i);
                                        break;
                                    }
                                }
                            })
                            .addOnFailureListener(e -> {
                                // Log or show a message
                            });
                },
                true);
        profileRecyclerView.setAdapter(myAdapter);

        loadMyAssignments();

        return view;
    }

    @Override
    public void onClick(View v) {
        if (v == ivAddTask) {
            final DialogPlus dialogPlus = DialogPlus.newDialog(getContext())
                    .setContentHolder(new ViewHolder(R.layout.update_popup))
                    .setExpanded(true, 1700)
                    .create();

            View dialogView = dialogPlus.getHolderView();

            Button btnUpdate = dialogView.findViewById(R.id.btnUpdate);
            EditText subtopic = dialogView.findViewById(R.id.subTopic);
            Spinner nameOfSubject = dialogView.findViewById(R.id.txtNameOfSubject);

            ArrayList<String> arrayList = new ArrayList<>();
            arrayList.add("Math");
            arrayList.add("English");
            arrayList.add("History");
            arrayList.add("Physics");
            arrayList.add("Computer Science");
            arrayList.add("Arabic");
            arrayList.add("Biology");
            arrayList.add("Literature");
            arrayList.add("Hebrew");
            arrayList.add("Citizenship");
            arrayList.add("Shelah");
            arrayList.add("Music");
            arrayList.add("Theater");
            arrayList.add("Art");
            arrayList.add("Chemistry");
            arrayList.add("Geography");
            arrayList.add("Philosophy");
            arrayList.add("New Media");
            arrayList.add("Law");
            arrayList.add("Finance");
            arrayList.add("Bible");
            String [] options = getResources().getStringArray(R.array.Spinner_items);
            ArrayAdapter<String> adapter = new ArrayAdapter<>(getContext(), android.R.layout.simple_spinner_item, options);
            adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
            nameOfSubject.setAdapter(adapter);

            CalendarView calendarView = dialogView.findViewById(R.id.calendarView);

            final Calendar calendar = Calendar.getInstance();

            calendarView.setOnDateChangeListener((view, year, month, dayOfMonth) -> {
                calendar.set(year, month, dayOfMonth);
            });


            btnUpdate.setOnClickListener(view -> {
                String nameOfSub = nameOfSubject.getSelectedItem().toString();
                String ST = subtopic.getText().toString();


                if (nameOfSub.isEmpty() || ST.isEmpty()) {
                    Log.e("Validation", "Subject or Subtopic cannot be empty.");
                    return;
                }

                uploadAssignment(nameOfSub, ST, calendar);
                dialogPlus.dismiss();
            });

            dialogPlus.show();
        }
    }

    public void uploadAssignment(String nameOfSub, String subtopic, Calendar calendar) {

        int day = calendar.get(Calendar.DAY_OF_MONTH);
        int month = calendar.get(Calendar.MONTH) + 1;
        int year = calendar.get(Calendar.YEAR);

        FirebaseFirestore db = FirebaseFirestore.getInstance();
        FirebaseAuth auth = FirebaseAuth.getInstance();

        String currentUserId = auth.getCurrentUser().getUid();
        String id = UUID.randomUUID().toString();

        Assignment assignment = new Assignment(nameOfSub, subtopic, day, month, year, currentUserId);

        db.collection("assignments").document(id).set(assignment)
                .addOnSuccessListener(aVoid -> {
                    Log.d("Firestore", "Assignment uploaded successfully!");
                })
                .addOnFailureListener(e -> {
                    Log.e("Firestore", "Error uploading assignment", e);
                });
    }
    private void loadMyAssignments() {
        String currentUserId = FirebaseAuth.getInstance().getCurrentUser().getUid();
        db.collection("assignments")
                .whereEqualTo("uploaderId", currentUserId)
                .get()
                .addOnSuccessListener(queryDocumentSnapshots -> {
                    myAssignments.clear();
                    for (QueryDocumentSnapshot doc : queryDocumentSnapshots) {
                        Assignment assignment = doc.toObject(Assignment.class);
                        assignment.setId(doc.getId());
                        myAssignments.add(assignment);
                    }
                    myAdapter.notifyDataSetChanged();
                })
                .addOnFailureListener(e -> Log.e("Firestore", "Error loading assignments", e));
    }
}





// ========================================
// UPDATED Register.java (Registration Activity)
package com.example.shareitapp;

import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;

import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.auth.AuthResult;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.auth.UserProfileChangeRequest;
import com.google.firebase.firestore.FirebaseFirestore;
import com.google.firebase.firestore.SetOptions;

import java.util.HashMap;
import java.util.Map;

public class Register extends AppCompatActivity implements View.OnClickListener{
    Button btnSubmit, btnLogIn;
    EditText etEmail, etPassword, etUserName;
    FirebaseAuth auth;
    FirebaseFirestore db;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_register);

        db = FirebaseFirestore.getInstance();
        etEmail = findViewById(R.id.email);
        etPassword = findViewById(R.id.password);
        etUserName = findViewById(R.id.userName);
        btnSubmit = findViewById(R.id.btnSubmit);
        btnLogIn = findViewById(R.id.btnToLogIn);
        btnSubmit.setOnClickListener(this);
        btnLogIn.setOnClickListener(this);
        auth = FirebaseAuth.getInstance();
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (auth.getCurrentUser() != null) {
            startActivity(new Intent(Register.this, MainActivity.class));
            finish();
        }
    }

    // ADD THIS METHOD
    private void saveUserToFirestore() {
        FirebaseUser user = FirebaseAuth.getInstance().getCurrentUser();
        if (user != null) {
            Map<String, Object> userData = new HashMap<>();
            userData.put("uid", user.getUid());
            userData.put("displayName", user.getDisplayName());
            userData.put("email", user.getEmail());

            FirebaseFirestore.getInstance()
                    .collection("users")
                    .document(user.getUid()) // Use UID as document ID
                    .set(userData, SetOptions.merge()) // Use merge to avoid overwriting existing data
                    .addOnSuccessListener(aVoid -> Log.d("Firestore", "User data saved successfully"))
                    .addOnFailureListener(e -> Log.e("Firestore", "Error saving user data", e));
        }
    }

    @Override
    public void onClick(View v) {
        if (v == btnLogIn) {
            startActivity(new Intent(this, Account.class));
        }
        if (v == btnSubmit) {
            String mail = etEmail.getText().toString();
            String pass = etPassword.getText().toString();
            String userName = etUserName.getText().toString();
            if (mail.isEmpty() || pass.isEmpty() || userName.isEmpty())
                Toast.makeText(this, "Fill all the fields" , Toast.LENGTH_LONG).show();
            else {
                auth.createUserWithEmailAndPassword(mail,pass).addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    public void onComplete( Task<AuthResult> task) {
                        if(task.isSuccessful()) {
                            FirebaseUser user = auth.getCurrentUser();
                            UserProfileChangeRequest profileUpdates = new UserProfileChangeRequest.Builder()
                                    .setDisplayName(userName)
                                    .build();

                            user.updateProfile(profileUpdates)
                                    .addOnCompleteListener(updateTask -> {
                                        if (updateTask.isSuccessful()) {
                                            Toast.makeText(Register.this, "Registration Successful", Toast.LENGTH_SHORT).show();

                                            // ADD THIS LINE - Save user data to Firestore after setting display name
                                            saveUserToFirestore();

                                            startActivity(new Intent(Register.this, MainActivity.class));
                                            finish();
                                        }
                                    });
                        } else {
                            Toast.makeText(Register.this, "Registration Failed", Toast.LENGTH_SHORT).show();
                        }
                    }
                });
            }
        }
    }
}






package com.example.shareitapp;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.example.shareitapp.YouTubeVideo;
import java.util.List;

public class VideoAdapter extends RecyclerView.Adapter<VideoAdapter.VideoViewHolder> {
    private Context context;
    private List<YouTubeVideo> videoList;

    public VideoAdapter(Context context, List<YouTubeVideo> videoList) {
        this.context = context;
        this.videoList = videoList;
    }

    @NonNull
    @Override
    public VideoViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.video_item, parent, false);
        return new VideoViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull VideoViewHolder holder, int position) {
        YouTubeVideo video = videoList.get(position);
        holder.subjectName.setText(video.getSubjectName());

        String videoId = extractYouTubeVideoId(video.getUrl());
        if (videoId != null) {
            String embedUrl = "https://www.youtube.com/embed/" + videoId;
            holder.webView.loadUrl(embedUrl);
        }
    }

    @Override
    public int getItemCount() {
        return videoList.size();
    }
    public static class VideoViewHolder extends RecyclerView.ViewHolder {
        WebView webView;
        TextView subjectName;

        public VideoViewHolder(@NonNull View itemView) {
            super(itemView);
            webView = itemView.findViewById(R.id.videoWebView);
            subjectName = itemView.findViewById(R.id.subjectName);

            webView.getSettings().setJavaScriptEnabled(true);
            webView.setWebViewClient(new WebViewClient());
        }
    }


    private String extractYouTubeVideoId(String url) {
        String regex = "^(?:https?://)?(?:www\\.)?(?:youtube\\.com/watch\\?v=|youtu\\.be/)([a-zA-Z0-9_-]{11})";
        java.util.regex.Pattern pattern = java.util.regex.Pattern.compile(regex);
        java.util.regex.Matcher matcher = pattern.matcher(url);
        return matcher.find() ? matcher.group(1) : null;
    }
}




package com.example.shareitapp;

public class YouTubeVideo {
    private String url;
    private String subjectName;

    public YouTubeVideo() {
        // Empty constructor for Firebase
    }

    public YouTubeVideo(String url, String subjectName) {
        this.url = url;
        this.subjectName = subjectName;
    }

    public String getUrl() {
        return url;
    }

    public String getSubjectName() {
        return subjectName;
    }
}


