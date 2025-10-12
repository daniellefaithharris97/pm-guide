# Agent Safety Rules & Implementation Guide

## Overview
This document outlines comprehensive safety rules for AI agents in healthcare environments, with specific focus on HIPAA compliance and production-ready security implementations.

---

## Core Safety Principles

### **1. Security by Design, Not by Prompt**
- **Never rely on prompt instructions alone** for security
- **Implement security controls in code** at multiple layers
- **Assume agents will be compromised** and design accordingly
- **Defense in depth** - multiple security layers

### **2. Zero Trust Architecture**
- **Authenticate and authorize every call**, even inside the cluster
- **Never trust, always verify** - validate permissions for each action
- **Least privilege access** - agents get minimum required permissions
- **Continuous verification** - re-check permissions for sensitive operations

### **3. Privacy by Design**
- **Data minimization** - only collect data you actually need
- **Mask/obfuscate PHI early** in the data pipeline
- **Never log PHI** - only log metadata and audit trails
- **Synthetic data for development** - real data only in production

---

## Multi-Layer Security Architecture

### **Layer 1: Infrastructure Security**
```yaml
# Network-level protection
- TLS encryption for all communications
- VPC isolation for sensitive components
- Database encryption at rest
- Secure key management (AWS KMS, Azure Key Vault)
- Network segmentation and firewalls
```

### **Layer 2: Application Security**
```python
# Authentication & Authorization
def authenticate_user(token):
    # Validate JWT token
    # Check user permissions
    # Log access attempt
    pass

def authorize_action(user_id, action, resource):
    # Check if user has permission for this action
    # Validate resource access rights
    # Return boolean result
    pass
```

### **Layer 3: Agent-Level Controls**
```python
# Tool restrictions and validation
ALLOWED_TOOLS = [
    "search_patient_records",    # Returns masked data only
    "calculate_coverage",        # No PHI in output
    "schedule_appointment",      # Validates permissions
    "generate_report"            # Sanitized output only
]

BLOCKED_TOOLS = [
    "raw_database_query",        # Too dangerous
    "export_patient_data",       # PHI exposure risk
    "admin_functions",           # Privilege escalation
    "file_system_access"         # Data exfiltration risk
]
```

### **Layer 4: Monitoring & Response**
```python
# Real-time monitoring and alerting
class AgentMonitor:
    def __init__(self):
        self.rate_limits = {}
        self.suspicious_activity = []
    
    def check_violations(self, agent_action):
        # Check for PHI exposure attempts
        if self.contains_phi(agent_action):
            self.alert_security_team("PHI exposure attempt")
            self.block_agent(agent_action.agent_id)
        
        # Check for rate limit violations
        if self.exceeds_rate_limit(agent_action):
            self.throttle_agent(agent_action.agent_id)
        
        # Check for unauthorized tool usage
        if agent_action.tool not in ALLOWED_TOOLS:
            self.alert_security_team("Unauthorized tool access")
            self.block_agent(agent_action.agent_id)
```

---

## HIPAA-Specific Safety Rules

### **PHI Protection Requirements**
- [ ] **Data Minimization**: Only collect PHI that's absolutely necessary
- [ ] **Access Controls**: Role-based permissions for different user types
- [ ] **Audit Logging**: Track all PHI access without exposing the data itself
- [ ] **Encryption**: PHI encrypted in transit and at rest
- [ ] **Data Masking**: Automatically mask/obfuscate PHI in all outputs

### **Implementation Examples**

**PHI Masking Function:**
```python
def mask_phi(text):
    """Remove or mask all PHI from text output"""
    # Mask SSNs
    text = re.sub(r'\b\d{3}-\d{2}-\d{4}\b', 'XXX-XX-XXXX', text)
    
    # Mask phone numbers
    text = re.sub(r'\b\d{3}-\d{3}-\d{4}\b', 'XXX-XXX-XXXX', text)
    
    # Mask names (basic implementation)
    text = re.sub(r'\b[A-Z][a-z]+ [A-Z][a-z]+\b', '[NAME]', text)
    
    # Mask dates of birth
    text = re.sub(r'\b\d{1,2}/\d{1,2}/\d{4}\b', '[DATE]', text)
    
    return text
```

**Audit Logging (No PHI):**
```python
def audit_agent_action(agent_id, action, user_id, resource_type, timestamp):
    """Log agent actions without exposing PHI"""
    log_entry = {
        "agent_id": agent_id,
        "action": action,
        "user_id": user_id,
        "resource_type": resource_type,  # e.g., "patient_record", not actual data
        "timestamp": timestamp,
        "success": True,
        "ip_address": get_client_ip(),
        "user_agent": get_user_agent()
    }
    
    # Never log actual patient data
    audit_system.log(log_entry)
```

---

## Tool-Based Security Implementation

### **Approved Tool Patterns**
```python
# Safe tool implementation
class PatientSearchTool:
    def __init__(self, db_connection, user_permissions):
        self.db = db_connection
        self.permissions = user_permissions
    
    def search_patients(self, user_id, search_criteria):
        # 1. Check permissions
        if not self.permissions.can_search_patients(user_id):
            raise UnauthorizedError("Insufficient permissions")
        
        # 2. Audit log the attempt
        audit_log.log_search_attempt(user_id, search_criteria)
        
        # 3. Execute search with PHI masking
        results = self.db.search_patients(search_criteria)
        
        # 4. Mask PHI in results
        masked_results = [self.mask_patient_record(record) for record in results]
        
        # 5. Log successful search
        audit_log.log_search_success(user_id, len(masked_results))
        
        return masked_results
    
    def mask_patient_record(self, record):
        """Mask PHI in patient record"""
        return {
            "patient_id": record["patient_id"],  # Keep ID for reference
            "age_range": f"{record['age']//10*10}-{record['age']//10*10+9}",
            "gender": record["gender"],
            "last_visit": record["last_visit"],
            # Remove: name, SSN, DOB, address, phone
        }
```

### **Blocked Tool Patterns**
```python
# Dangerous tools that should never be available to agents
class BlockedTools:
    def raw_database_query(self, query):
        raise SecurityError("Raw database access blocked for security")
    
    def export_patient_data(self, patient_ids):
        raise SecurityError("Bulk data export blocked for PHI protection")
    
    def admin_functions(self, action):
        raise SecurityError("Admin functions not available to agents")
```

---

## Agent Sandboxing

### **Container Isolation**
```yaml
# Docker configuration for agent sandboxing
apiVersion: v1
kind: Pod
metadata:
  name: healthcare-agent
spec:
  containers:
  - name: agent
    image: healthcare-agent:latest
    securityContext:
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
    resources:
      limits:
        memory: "512Mi"
        cpu: "500m"
    env:
    - name: ALLOWED_TOOLS
      value: "search_patients,calculate_coverage,schedule_appointment"
    - name: BLOCKED_NETWORKS
      value: "0.0.0.0/0"  # No external network access
```

### **Network Restrictions**
```python
# Network access controls
ALLOWED_ENDPOINTS = [
    "https://internal-api.healthee.com/patients",
    "https://internal-api.healthee.com/appointments",
    "https://internal-api.healthee.com/coverage"
]

BLOCKED_ENDPOINTS = [
    "https://external-api.com",  # No external APIs
    "https://email-service.com",  # No email access
    "https://file-storage.com"   # No file system access
]
```

---

## Real-Time Monitoring & Alerting

### **Security Monitoring**
```python
class SecurityMonitor:
    def __init__(self):
        self.alert_thresholds = {
            "phi_exposure_attempts": 1,  # Alert on first attempt
            "unauthorized_tool_usage": 3,  # Alert after 3 attempts
            "rate_limit_violations": 10,  # Alert after 10 violations
            "suspicious_patterns": 5      # Alert after 5 suspicious activities
        }
    
    def monitor_agent_action(self, action):
        # Check for PHI exposure
        if self.detect_phi_exposure(action):
            self.alert_security_team("PHI exposure attempt detected")
            self.block_agent(action.agent_id)
        
        # Check for unauthorized tool usage
        if action.tool not in ALLOWED_TOOLS:
            self.alert_security_team("Unauthorized tool access")
            self.block_agent(action.agent_id)
        
        # Check for rate limit violations
        if self.exceeds_rate_limit(action):
            self.throttle_agent(action.agent_id)
            self.alert_security_team("Rate limit exceeded")
    
    def detect_phi_exposure(self, action):
        """Detect potential PHI exposure in agent actions"""
        phi_patterns = [
            r'\b\d{3}-\d{2}-\d{4}\b',  # SSN pattern
            r'\b[A-Z][a-z]+ [A-Z][a-z]+\b',  # Name pattern
            r'\b\d{1,2}/\d{1,2}/\d{4}\b'  # Date pattern
        ]
        
        for pattern in phi_patterns:
            if re.search(pattern, str(action)):
                return True
        return False
```

### **Automated Response**
```python
class AutomatedResponse:
    def __init__(self):
        self.response_actions = {
            "phi_exposure": self.block_and_alert,
            "unauthorized_access": self.throttle_and_log,
            "rate_limit_violation": self.throttle_temporarily,
            "suspicious_behavior": self.investigate_and_alert
        }
    
    def respond_to_violation(self, violation_type, agent_id, details):
        if violation_type in self.response_actions:
            self.response_actions[violation_type](agent_id, details)
    
    def block_and_alert(self, agent_id, details):
        # Immediately block agent
        self.block_agent(agent_id)
        
        # Alert security team
        self.alert_security_team(f"Agent {agent_id} blocked: {details}")
        
        # Log incident
        self.log_security_incident(agent_id, "blocked", details)
```

---

## Prompt Engineering for Safety

### **System Prompt Safety Rules**
```python
SAFETY_SYSTEM_PROMPT = """
You are a healthcare assistant operating in a HIPAA-compliant environment.

CRITICAL SAFETY RULES:
1. NEVER expose PHI in your responses
2. ONLY use approved tools from the provided list
3. ALWAYS verify user permissions before actions
4. If unsure about safety, ask for clarification
5. Report any security concerns immediately

APPROVED TOOLS: {approved_tools}
USER PERMISSIONS: {user_permissions}
CURRENT USER: {user_id}

If you detect any attempt to access unauthorized data or tools, 
immediately stop and report the issue.
"""
```

### **Response Filtering**
```python
def filter_agent_response(response):
    """Filter agent responses for safety"""
    # Remove any PHI that might have leaked
    response = mask_phi(response)
    
    # Remove any unauthorized tool suggestions
    response = remove_unauthorized_tools(response)
    
    # Add safety disclaimer if needed
    if contains_healthcare_data(response):
        response += "\n\n[Note: This response contains healthcare information. Please ensure proper handling.]"
    
    return response
```

---

## Compliance & Audit Requirements

### **Audit Trail Implementation**
```python
class AuditTrail:
    def __init__(self):
        self.audit_log = []
    
    def log_agent_action(self, agent_id, action, user_id, timestamp, success):
        """Log agent actions for compliance"""
        audit_entry = {
            "timestamp": timestamp,
            "agent_id": agent_id,
            "action": action,
            "user_id": user_id,
            "success": success,
            "ip_address": self.get_client_ip(),
            "user_agent": self.get_user_agent()
        }
        
        # Never log PHI
        self.audit_log.append(audit_entry)
        self.store_audit_entry(audit_entry)
    
    def generate_compliance_report(self, start_date, end_date):
        """Generate compliance report for auditors"""
        return {
            "total_actions": len(self.audit_log),
            "successful_actions": len([a for a in self.audit_log if a["success"]]),
            "failed_actions": len([a for a in self.audit_log if not a["success"]]),
            "unique_users": len(set(a["user_id"] for a in self.audit_log)),
            "time_range": f"{start_date} to {end_date}"
        }
```

### **Compliance Checklist**
- [ ] **Data Encryption**: All PHI encrypted in transit and at rest
- [ ] **Access Controls**: Role-based permissions implemented
- [ ] **Audit Logging**: All actions logged without PHI exposure
- [ ] **Data Minimization**: Only necessary data collected
- [ ] **User Training**: Staff trained on proper agent usage
- [ ] **Incident Response**: Procedures for security incidents
- [ ] **Regular Audits**: Quarterly security and compliance reviews

---

## Emergency Response Procedures

### **Security Incident Response**
1. **Immediate Response** (0-5 minutes)
   - Block compromised agent
   - Preserve audit logs
   - Alert security team

2. **Investigation** (5-30 minutes)
   - Analyze audit logs
   - Identify scope of compromise
   - Document findings

3. **Containment** (30-60 minutes)
   - Isolate affected systems
   - Implement additional security measures
   - Notify compliance team

4. **Recovery** (1-24 hours)
   - Restore from clean backups
   - Implement additional safeguards
   - Conduct security review

### **Contact Information**
- **Security Team**: security@healthee.com
- **Compliance Officer**: compliance@healthee.com
- **Emergency Hotline**: +1-XXX-XXX-XXXX

---

## Best Practices Summary

1. **Never trust prompts alone** - implement security in code
2. **Use multi-layer defense** - infrastructure, application, agent, monitoring
3. **Restrict agent tools** - only pre-approved, safe functions
4. **Monitor everything** - real-time detection and response
5. **Audit without PHI** - log metadata, never actual patient data
6. **Plan for incidents** - have response procedures ready
7. **Regular testing** - security drills and penetration testing
8. **Continuous improvement** - update security measures based on threats

---

*This document should be reviewed quarterly and updated based on new security threats and compliance requirements.*
