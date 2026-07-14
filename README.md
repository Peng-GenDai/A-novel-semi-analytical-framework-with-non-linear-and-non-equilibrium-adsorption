# A-novel-semi-analytical-framework-with-non-linear-and-non-equilibrium-adsorption
This code can predict the transport of reactive contaminant in porous media, taking into account non-equilibrium, non-linear adsorption.
function WR_NLNE_AllFigures_Master_v3(figList)
% =========================================================================
% WR_NLNE_AllFigures_Master_.m
% -------------------------------------------------------------------------
% One-file MATLAB code for the numerical figures in the revised manuscript.
%
% Revised figure numbering:
%   Fig. 4  Laboratory flow-chamber validation (Jang et al. 2026)
%   Fig. 5  Two-layer analytical benchmark (Li & Cleall-type case)
%   Fig. 6  Three-layer benchmark with source decay and degradation
%   Fig. 7  FDM benchmark with nonlinear Freundlich sorption and Robin outlet
%   Fig. 8  FEM benchmark scenarios: Freundlich and Langmuir, two inlet BCs
%   Fig. 9  Mechanistic comparison: NENL, ENL, NEL, EL
%   Fig.10  Non-equilibrium parameters: k_s and f_i
%   Fig.11  Nonlinear Freundlich exponent N_i and apparent retardation
%
% Notes:
%   1) Fig. 1-Fig. 3 are conceptual/algorithmic diagrams and are not generated here.
%   2) Fig. 4 includes digitized laboratory points and exports Origin-ready CSV files.
%   3) Original numerical figures after Fig. 4 have been shifted by +1 in numbering.
%   4) For the nonlinear formulation, the tangent/secant separation is retained:
%      accumulation uses tangent coefficients, while kinetic/decay terms use secant coefficients.
% =========================================================================
clc; close all;
if nargin == 0
    figList = {'Fig4','Fig5','Fig6','Fig7','Fig8','Fig9','Fig10','Fig11'};
end
if ischar(figList) || isstring(figList)
    figList = cellstr(figList);
end

fprintf('\n==============================================================\n');
fprintf(' WR_NLNE_AllFigures_Master_v3 started.\n');
fprintf('==============================================================\n');

for ii = 1:numel(figList)
    tag = lower(strrep(figList{ii},' ',''));
    switch tag
        case {'fig4','4'}
            make_Fig4_Experiment('fast');
        case {'fig5','5'}
            make_Fig5_TwoLayer();
        case {'fig6','6'}
            make_Fig6_ThreeLayer();
        case {'fig7','7'}
            make_Fig7_FDM();
        case {'fig8','8'}
            make_Fig8_FEM();
        case {'fig9','9'}
            make_Fig9_Mechanistic();
        case {'fig10','10'}
            make_Fig10_NEParams();
        case {'fig11','11'}
            make_Fig11_FreundlichExponent();
        otherwise
            warning('Unknown figure tag: %s. Skipped.', figList{ii});
    end
end
fprintf('\nAll requested figures have been generated.\n');
end

function make_Fig4_Experiment(mode)
% =========================================================================
% Fig. 4 experimental profile validation using the user's 8-column digitized data.
%
% Data mapping supplied by the user:
%   columns 1-2: Fig.4b T-clay red points, 1.8 d, [C_mgL, z_mm]
%   columns 3-4: Fig.4b T-clay purple points, 7.5 d, [C_mgL, z_mm]
%   columns 5-6: Fig.4d B-clay red points, 1.8 d, [C_mgL, z_mm]
%   columns 7-8: Fig.4d B-clay purple points, 7.5 d, [C_mgL, z_mm]
%
% Plot meaning:
%   open markers : experimental points
%   dashed lines : constant equivalent retardation model, R_eq = 1.2
%   solid lines  : present nonlinear Langmuir semi-analytical model
%
% Usage:
%   WR_NLNE_AllFigures_Master_v3(Fig4)
%   WR_NLNE_AllFigures_Master_v3(Fig4)('fast')
%   WR_NLNE_AllFigures_Master_v3(Fig4)('final')
% =========================================================================

if nargin == 0
    mode = 'fast';
end
close all; clc;

%% 1. Digitized experimental points [C_mgL, z_mm]
T_red_1p8d_Cz = [
    475.70316 0.40423
    421.69763 0.81684
    397.56330 1.22945
    376.19847 1.63120
    346.52511 2.01124
    340.39261 2.43470
    319.42343 2.90160
    299.24554 3.32507
    279.85895 3.71596
    259.08759 4.12857
    227.83164 4.53032
    217.14923 4.93207
    204.09295 5.30125
    181.14555 5.72472
    170.46314 6.12647
    159.38508 6.53907
    136.23985 6.91911
    122.59011 7.29914
    108.34689 7.79862
    101.81875 8.22208
    95.88408 8.61298
    78.67352 8.99301
    52.95661 9.40562
    34.75694 9.80737
    21.10719 10.17655
    25.26147 10.58916
    24.07453 11.01262
    17.34857 11.40351
    12.79865 11.81612
    18.73332 12.20702
    23.48106 12.67392
    23.48106 13.09738
    23.87671 13.50999
    18.13986 13.90088
    16.95292 14.34607
    12.00736 14.71524
    11.21607 15.14957
    13.58994 15.51874
    10.42478 15.90964
    5.67704 16.28967
    8.05091 16.73485
    14.38123 17.07146
    12.99647 17.57093
    11.80954 18.00525
    4.88575 18.42872
    12.60083 18.78704
];

T_purple_7p5d_Cz = [
    480.05525 1.21147
    456.31656 1.58100
    429.41271 1.99401
    414.18038 2.40701
    419.32376 2.90696
    401.91539 3.30910
    383.51790 3.71123
    364.52695 4.11337
    337.02963 4.91764
    334.65576 4.49377
    315.07134 5.30891
    303.20199 5.68930
    301.02595 6.11318
    275.50685 6.50444
    253.74638 6.88484
    242.86615 7.30872
    234.95325 7.79780
    230.00769 8.21080
    233.37067 8.61294
    218.53399 9.01507
    212.00585 9.40634
    205.47771 9.80848
    197.96046 10.21061
    194.99312 10.61275
    177.98039 10.99315
    164.52846 11.40615
    150.08742 11.79742
    154.43952 12.21042
    154.83516 12.69950
    152.46129 13.12338
    134.85510 13.50377
    125.75526 13.90591
    120.21624 14.30805
    106.36867 14.69931
    102.61004 15.10145
    96.47754 15.49271
    85.39949 15.90572
    72.73885 16.30785
    80.65175 16.69912
    81.64086 17.11212
    76.49748 17.62295
    65.61724 18.00334
    51.37403 18.40548
];

B_red_1p8d_Cz = [
    475.10969 0.45983
    458.69043 0.90936
    406.46531 1.34911
    366.70300 1.78887
    348.30551 2.27748
    316.45610 2.71724
    296.87167 3.16676
    274.12209 3.59675
    251.17469 4.05604
    222.29262 4.51534
    210.22545 4.95510
    193.80618 5.39485
    177.58474 5.85415
    160.76984 6.28413
    144.54840 6.74343
    123.38140 7.18318
    111.11640 7.64248
    102.80786 8.09201
    87.57553 8.54154
    69.77151 8.98129
    51.37403 9.44059
    30.60267 9.91943
    16.75510 10.34941
    9.83132 10.76962
    11.80954 11.26801
    7.25962 11.67845
    7.06180 12.17684
    6.46833 12.61659
    6.27051 13.06612
    6.86398 13.53519
    0.92931 13.93585
    5.67704 14.36583
    5.08358 14.84468
    3.89664 15.30398
    6.46833 15.74373
    4.09446 16.20303
    4.88575 16.65255
    3.10535 17.08254
    3.10535 17.54183
    0.00000 17.98159
];

B_purple_7p5d_Cz = [
    493.70500 0.00000
    486.38557 0.89067
    452.75576 1.35041
    440.09512 1.81015
    419.32376 2.26988
    405.08055 2.71006
    382.72661 3.16980
    368.48340 3.61975
    356.02058 4.05993
    344.54688 4.50989
    339.79914 4.97941
    324.96246 5.40002
    307.35626 5.84998
    298.05861 6.29993
    288.95878 6.75967
    273.92427 7.21941
    267.98960 7.66937
    261.65928 8.10954
    246.82260 8.53994
    229.01858 8.98989
    217.54488 9.46919
    205.87335 9.88981
    191.82796 10.36911
    184.90417 10.78972
    186.28893 11.26902
    179.95861 11.71898
    176.99128 12.14937
    175.80434 12.60911
    174.02394 13.05907
    174.61741 13.51881
    166.90233 13.97855
    172.24354 14.41872
    170.46314 14.84911
    166.11104 15.29907
    169.67185 15.74903
    166.90233 16.21855
    166.90233 16.65872
    164.52846 17.10868
    160.96766 17.55864
    158.39597 17.99881
];

% Convert to [z_mm, C_mgL] for interpolation and plotting.
Dat.T_red   = sortrows([T_red_1p8d_Cz(:,2),   T_red_1p8d_Cz(:,1)],   1);
Dat.T_purple = sortrows([T_purple_7p5d_Cz(:,2), T_purple_7p5d_Cz(:,1)], 1);
Dat.B_red   = sortrows([B_red_1p8d_Cz(:,2),   B_red_1p8d_Cz(:,1)],   1);
Dat.B_purple = sortrows([B_purple_7p5d_Cz(:,2), B_purple_7p5d_Cz(:,1)], 1);

%% 2. Physical parameters for the reduced experimental validation
par = struct();
par.C0    = 500;            % mg/L
par.D0    = 3.8e-5;         % m^2/day
par.tau   = 0.28;           % -
par.Daq   = par.tau*par.D0; % m^2/day
par.n     = 0.60;           % -
par.rho_b = 1.20;           % kg/L = 1.2 g/mL
par.R_eq  = 1.20;           % constant equivalent retardation factor
par.LT    = 20e-3;          % m, T-clay thickness
par.LB    = 18e-3;          % m, B-clay thickness

% Langmuir parameter setting for the final validation.
% The batch-fitted parameters reported by Jang et al. are:
%     Qm_batch = 0.210 mg/g, KL_batch = 0.130 L/mg.
% Directly using these two batch parameters gives a very large low-concentration
% retardation factor, R(0) = 55.6, and makes the Laplace-Picard validation
% numerically unstable for the early profiles.
%
% Therefore, this final validation uses an effective chamber-scale Langmuir
% parameter set constrained by the equivalent retardation factor R_eq = 1.2:
%     1000*Qm_eff*KL_eff = (R_eq - 1)*n/rho_b.
% To keep the Langmuir affinity coefficient consistent with the batch fitting,
% KL_eff is set equal to the reported KL_batch = 0.130 L/mg, while Qm_eff is
% back-calculated from R_eq.
Qm_batch = 0.210;                            % mg/g, batch-fitted reference value
KL_batch = 0.130;                            % L/mg, batch-fitted reference value

Kd_eq  = (par.R_eq - 1)*par.n/par.rho_b;     % L/kg, from R_eq = 1.2
KL_eff = KL_batch;                           % L/mg, use the reported Langmuir affinity
Qm_eff = Kd_eq/(1000*KL_eff);                % mg/g, effective chamber-scale capacity

par.Qm = Qm_eff;                             % = 0.00076923 mg/g
par.KL = KL_eff;                             % = 0.130 L/mg

%% 3. Semi-analytical algorithm settings
switch lower(mode)
    case 'final'
        par.Ncell     = 200;
        par.Mtalbot   = 64;
        par.maxPicard = 50;
        par.tolPicard = 1e-4;
        par.relax     = 0.80;
        par.nHomo     = 5;
    otherwise
        par.Ncell     = 60;
        par.Mtalbot   = 32;
        par.maxPicard = 20;
        par.tolPicard = 2e-4;
        par.relax     = 0.70;
        par.nHomo     = 3;
end
par.Cmin = 1e-9;

%% 4. Compute profiles
Times = [1.8, 7.5];       % red and purple points
zT_mm = linspace(0,20,401)';
zB_mm = linspace(0,18,361)';
zT = zT_mm/1000;
zB = zB_mm/1000;

C_T_lin = zeros(numel(zT),2);
C_T_lan = zeros(numel(zT),2);
C_B_lin = zeros(numel(zB),2);
C_B_lan = zeros(numel(zB),2);

fprintf('>>> Computing Fig.4 using user-supplied points (%s mode) ...\n', mode);
for k = 1:2
    t = Times(k);
    C_T_lin(:,k) = profile_linear(t,'Tclay',zT,par);
    C_T_lan(:,k) = profile_langmuir(t,'Tclay',zT,par);
    C_B_lin(:,k) = profile_linear(t,'Bclay',zB,par);
    C_B_lan(:,k) = profile_langmuir(t,'Bclay',zB,par);
end

%% 5. Goodness-of-fit statistics
Met = struct();
% Use a safe interpolation routine for metrics. A few digitized points may
% fall slightly outside the plotting domain because of image digitization
% uncertainty, for example z = 18.00113 mm for B-clay. Those locations are
% clipped to the model domain before interpolation. NaN values are removed
% before calculating NSE and RMSE.
Met.T_red_lin    = metrics_at_exp(Dat.T_red(:,2),    Dat.T_red(:,1),    zT_mm, C_T_lin(:,1));
Met.T_red_lan    = metrics_at_exp(Dat.T_red(:,2),    Dat.T_red(:,1),    zT_mm, C_T_lan(:,1));
Met.T_purple_lin = metrics_at_exp(Dat.T_purple(:,2), Dat.T_purple(:,1), zT_mm, C_T_lin(:,2));
Met.T_purple_lan = metrics_at_exp(Dat.T_purple(:,2), Dat.T_purple(:,1), zT_mm, C_T_lan(:,2));
Met.B_red_lin    = metrics_at_exp(Dat.B_red(:,2),    Dat.B_red(:,1),    zB_mm, C_B_lin(:,1));
Met.B_red_lan    = metrics_at_exp(Dat.B_red(:,2),    Dat.B_red(:,1),    zB_mm, C_B_lan(:,1));
Met.B_purple_lin = metrics_at_exp(Dat.B_purple(:,2), Dat.B_purple(:,1), zB_mm, C_B_lin(:,2));
Met.B_purple_lan = metrics_at_exp(Dat.B_purple(:,2), Dat.B_purple(:,1), zB_mm, C_B_lan(:,2));

fprintf('\n==================== Fig.4 metrics ====================\n');
fprintf('T-clay 1.8 d | constant R: NSE = %.4f, RMSE = %.3f | present: NSE = %.4f, RMSE = %.3f\n', ...
    Met.T_red_lin.NSE, Met.T_red_lin.RMSE, Met.T_red_lan.NSE, Met.T_red_lan.RMSE);
fprintf('T-clay 7.5 d | constant R: NSE = %.4f, RMSE = %.3f | present: NSE = %.4f, RMSE = %.3f\n', ...
    Met.T_purple_lin.NSE, Met.T_purple_lin.RMSE, Met.T_purple_lan.NSE, Met.T_purple_lan.RMSE);
fprintf('B-clay 1.8 d | constant R: NSE = %.4f, RMSE = %.3f | present: NSE = %.4f, RMSE = %.3f\n', ...
    Met.B_red_lin.NSE, Met.B_red_lin.RMSE, Met.B_red_lan.NSE, Met.B_red_lan.RMSE);
fprintf('B-clay 7.5 d | constant R: NSE = %.4f, RMSE = %.3f | present: NSE = %.4f, RMSE = %.3f\n', ...
    Met.B_purple_lin.NSE, Met.B_purple_lin.RMSE, Met.B_purple_lan.NSE, Met.B_purple_lan.RMSE);
fprintf('========================================================\n');

%% 6. Export to workspace
Origin_Fig4_Tclay = [zT_mm, C_T_lin(:,1), C_T_lan(:,1), C_T_lin(:,2), C_T_lan(:,2)];
Origin_Fig4_Bclay = [zB_mm, C_B_lin(:,1), C_B_lan(:,1), C_B_lin(:,2), C_B_lan(:,2)];
Fig4_ModelParams = table(par.C0,par.D0,par.tau,par.n,par.rho_b,par.R_eq,par.Qm,par.KL, ...
    'VariableNames',{'C0_mgL','D0_m2day','tau','n','rho_b_kgL','R_eq','Qm_mgg','KL_Lmg'});
assignin('base','Origin_Fig4_Tclay',Origin_Fig4_Tclay);
assignin('base','Origin_Fig4_Bclay',Origin_Fig4_Bclay);
assignin('base','DigitizedPoints_Tclay_Red_zC',Dat.T_red);
assignin('base','DigitizedPoints_Tclay_Purple_zC',Dat.T_purple);
assignin('base','DigitizedPoints_Bclay_Red_zC',Dat.B_red);
assignin('base','DigitizedPoints_Bclay_Purple_zC',Dat.B_purple);
assignin('base','Fig4_Metrics',Met);
assignin('base','Fig4_ModelParams',Fig4_ModelParams);


%% 7. Export CSV files for Origin plotting
% Each CSV uses paired X-Y columns. In Origin, set C columns as X and z
% columns as Y, then plot line+symbol/line curves.
OriginDir = fullfile(pwd,'Fig4_Origin_Data');
if ~exist(OriginDir,'dir'), mkdir(OriginDir); end

Origin_Fig4a_Tclay_Table = make_origin_table(zT_mm, C_T_lin, C_T_lan, Dat.T_red, Dat.T_purple);
Origin_Fig4b_Bclay_Table = make_origin_table(zB_mm, C_B_lin, C_B_lan, Dat.B_red, Dat.B_purple);

MetricNames = {'T_clay_1p8d'; 'T_clay_7p5d'; 'B_clay_1p8d'; 'B_clay_7p5d'};
NSE_ConstantR = [Met.T_red_lin.NSE; Met.T_purple_lin.NSE; Met.B_red_lin.NSE; Met.B_purple_lin.NSE];
RMSE_ConstantR = [Met.T_red_lin.RMSE; Met.T_purple_lin.RMSE; Met.B_red_lin.RMSE; Met.B_purple_lin.RMSE];
NSE_Present = [Met.T_red_lan.NSE; Met.T_purple_lan.NSE; Met.B_red_lan.NSE; Met.B_purple_lan.NSE];
RMSE_Present = [Met.T_red_lan.RMSE; Met.T_purple_lan.RMSE; Met.B_red_lan.RMSE; Met.B_purple_lan.RMSE];
Fig4_Metrics_Table = table(MetricNames, NSE_ConstantR, RMSE_ConstantR, NSE_Present, RMSE_Present);

writetable(Origin_Fig4a_Tclay_Table, fullfile(OriginDir,'Fig4a_Tclay_Origin.csv'));
writetable(Origin_Fig4b_Bclay_Table, fullfile(OriginDir,'Fig4b_Bclay_Origin.csv'));
writetable(Fig4_Metrics_Table, fullfile(OriginDir,'Fig4_Metrics.csv'));
writetable(Fig4_ModelParams, fullfile(OriginDir,'Fig4_ModelParams.csv'));

assignin('base','Origin_Fig4a_Tclay_Table',Origin_Fig4a_Tclay_Table);
assignin('base','Origin_Fig4b_Bclay_Table',Origin_Fig4b_Bclay_Table);
assignin('base','Fig4_Metrics_Table',Fig4_Metrics_Table);

fprintf('\n>>> Origin-ready CSV files have been written to folder:\n    %s\n', OriginDir);
fprintf('    - Fig4a_Tclay_Origin.csv\n');
fprintf('    - Fig4b_Bclay_Origin.csv\n');
fprintf('    - Fig4_Metrics.csv\n');
fprintf('    - Fig4_ModelParams.csv\n');

%% 8. Plot
cr = [0.85 0.15 0.12];    % red for 1.8 d
cg = [0.45 0.10 0.75];    % purple for 7.5 d

figure('Color','w','Position',[100 100 1080 450],'Name','Fig. 4 experimental validation');
tiledlayout(1,2,'Padding','compact','TileSpacing','compact');

% T-clay
nexttile; hold on; box on;
plot(C_T_lan(:,1), zT_mm, '-',  'Color', cr, 'LineWidth', 2.2, 'DisplayName','Present model, 1.8 d');
plot(C_T_lin(:,1), zT_mm, '--', 'Color', cr, 'LineWidth', 1.8, 'DisplayName','Constant R, 1.8 d');
plot(Dat.T_red(:,2), Dat.T_red(:,1), 'o', 'Color', cr, 'MarkerFaceColor','w', 'MarkerSize',6.5, 'LineWidth',1.1, 'DisplayName','Experiment, 1.8 d');
plot(C_T_lan(:,2), zT_mm, '-',  'Color', cg, 'LineWidth', 2.2, 'DisplayName','Present model, 7.5 d');
plot(C_T_lin(:,2), zT_mm, '--', 'Color', cg, 'LineWidth', 1.8, 'DisplayName','Constant R, 7.5 d');
plot(Dat.T_purple(:,2), Dat.T_purple(:,1), 's', 'Color', cg, 'MarkerFaceColor','w', 'MarkerSize',6.3, 'LineWidth',1.1, 'DisplayName','Experiment, 7.5 d');
set(gca,'YDir','reverse','FontName','Times New Roman','FontSize',12,'LineWidth',1.1,'TickDir','in','XAxisLocation','top');
xlabel('New Coccine concentration (mg/L)','FontName','Times New Roman','FontSize',13);
ylabel('Penetration distance in T-clay (mm)','FontName','Times New Roman','FontSize',13);
title('(a) T-clay','FontName','Times New Roman','FontSize',13);
xlim([0 500]); ylim([0 20]);
legend('Location','southeast','Box','off','FontSize',9);

% B-clay
nexttile; hold on; box on;
plot(C_B_lan(:,1), zB_mm, '-',  'Color', cr, 'LineWidth', 2.2, 'DisplayName','Present model, 1.8 d');
plot(C_B_lin(:,1), zB_mm, '--', 'Color', cr, 'LineWidth', 1.8, 'DisplayName','Constant R, 1.8 d');
plot(Dat.B_red(:,2), Dat.B_red(:,1), 'o', 'Color', cr, 'MarkerFaceColor','w', 'MarkerSize',6.5, 'LineWidth',1.1, 'DisplayName','Experiment, 1.8 d');
plot(C_B_lan(:,2), zB_mm, '-',  'Color', cg, 'LineWidth', 2.2, 'DisplayName','Present model, 7.5 d');
plot(C_B_lin(:,2), zB_mm, '--', 'Color', cg, 'LineWidth', 1.8, 'DisplayName','Constant R, 7.5 d');
plot(Dat.B_purple(:,2), Dat.B_purple(:,1), 's', 'Color', cg, 'MarkerFaceColor','w', 'MarkerSize',6.3, 'LineWidth',1.1, 'DisplayName','Experiment, 7.5 d');
set(gca,'YDir','reverse','FontName','Times New Roman','FontSize',12,'LineWidth',1.1,'TickDir','in','XAxisLocation','top');
xlabel('New Coccine concentration (mg/L)','FontName','Times New Roman','FontSize',13);
ylabel('Penetration distance in B-clay (mm)','FontName','Times New Roman','FontSize',13);
title('(b) B-clay','FontName','Times New Roman','FontSize',13);
xlim([0 500]); ylim([0 18]);
legend('Location','southeast','Box','off','FontSize',9);

end

%% ========================================================================
% Linear model
% ========================================================================
function C = profile_linear(t, clayType, zEval, par)
Rvec = par.R_eq*ones(par.Ncell,1);
C = invert_profile(t, clayType, Rvec, zEval, par);
C = max(real(C),0);
end

%% ========================================================================
% Present nonlinear Langmuir model
% ========================================================================
function C = profile_langmuir(t, clayType, zEval, par)
switch lower(clayType)
    case 'tclay'
        L = par.LT;
    case 'bclay'
        L = par.LB;
    otherwise
        error('Unknown clayType.');
end

zMid = linspace(L/(2*par.Ncell), L-L/(2*par.Ncell), par.Ncell)';
Rold = par.R_eq*ones(par.Ncell,1);
Cold = invert_profile(t, clayType, Rold, zMid, par);
Cold = max(real(Cold),0);

for ih = 1:par.nHomo
    eta = ih/par.nHomo;
    for ip = 1:par.maxPicard
        Rtar = (1-eta)*par.R_eq + eta*R_langmuir_tangent(Cold, par);
        Rnew = exp((1-par.relax)*log(max(Rold,1e-10)) + par.relax*log(max(Rtar,1e-10)));
        Cnew = invert_profile(t, clayType, Rnew, zMid, par);
        Cnew = max(real(Cnew),0);
        err = max(abs(Cnew-Cold)./(abs(Cnew)+par.Cmin));
        Cold = Cnew;
        Rold = Rnew;
        if err < par.tolPicard
            break;
        end
    end
end

C = invert_profile(t, clayType, Rold, zEval, par);
C = max(real(C),0);
end

function R = R_langmuir_tangent(C, par)
C = max(real(C),0);
Ktan = 1000*par.Qm*par.KL./(1 + par.KL*C).^2;   % L/kg
R = 1 + (par.rho_b/par.n)*Ktan;
end

%% ========================================================================
% Talbot inversion and Laplace-domain transfer matrix
% ========================================================================
function C = invert_profile(t, clayType, Rvec, zEval, par)
C = talbot_inversion(@(s) laplace_profile(s, clayType, Rvec, zEval, par), t, par.Mtalbot);
end

function f = talbot_inversion(Fs, t, M)
if t <= 0
    tmp = Fs(1);
    f = zeros(size(tmp));
    return;
end
fsum = [];
for k = 0:M-1
    theta = k*pi/M;
    if k == 0
        s = 2*M/(5*t);
        weight = 0.5*exp(s*t);
    else
        cotth = cot(theta);
        s = 2*M/(5*t)*(theta*cotth + 1i*theta);
        sigma = theta + (theta*cotth - 1)*cotth;
        weight = exp(s*t)*(1 + 1i*sigma);
    end
    Fval = Fs(s);
    if isempty(fsum)
        fsum = zeros(size(Fval));
    end
    fsum = fsum + weight.*Fval;
end
f = real(2/(5*t)*fsum);
end

function Cbar = laplace_profile(s, clayType, Rvec, zEval, par)
[~, segL] = clay_geometry(clayType, numel(Rvec), par);
Csrc_bar = par.C0/s;  % step boundary source
D = par.Daq;
A = total_transfer_matrix(s, Rvec, segL, D);

switch lower(clayType)
    case 'tclay'
        % z = 0 : B-sand/T-clay interface, C = C0
        % z = L : T-sand/T-clay interface, C = 0
        Cleft  = Csrc_bar;
        Cright = 0;
        Gleft  = (Cright - A(1,1)*Cleft)/A(1,2);
    case 'bclay'
        % z = 0 : B-sand/B-clay interface, C = C0
        % z = L : no-flux bottom boundary
        Cleft = Csrc_bar;
        Gleft = -A(2,1)*Cleft/A(2,2);
    otherwise
        error('Unknown clayType.');
end

Cbar = zeros(size(zEval));
for i = 1:numel(zEval)
    Ai = partial_transfer_matrix(s, Rvec, segL, D, zEval(i));
    state = Ai*[Cleft; Gleft];
    Cbar(i) = state(1);
end
end

function [Ltot, segL] = clay_geometry(clayType, N, par)
switch lower(clayType)
    case 'tclay'
        Ltot = par.LT;
    case 'bclay'
        Ltot = par.LB;
    otherwise
        error('Unknown clayType.');
end
segL = (Ltot/N)*ones(N,1);
end

function A = total_transfer_matrix(s, Rvec, segL, D)
A = eye(2);
for j = 1:numel(Rvec)
    A = layer_transfer(s, Rvec(j), segL(j), D) * A;
end
end

function A = partial_transfer_matrix(s, Rvec, segL, D, z)
A = eye(2);
xRemain = z;
for j = 1:numel(Rvec)
    if xRemain <= 0, break; end
    dx = min(segL(j), xRemain);
    A = layer_transfer(s, Rvec(j), dx, D) * A;
    xRemain = xRemain - dx;
end
end

function T = layer_transfer(s, R, L, D)
q  = sqrt(R*s/D);
qL = q*L;
cq = cosh(qL);
sq = sinh(qL);
T = [cq, sq/(D*q); D*q*sq, cq];
end


%% ========================================================================
% Build Origin-ready wide table with paired X-Y columns
% ========================================================================
function T = make_origin_table(z_mm, C_lin, C_lan, Exp_red_zC, Exp_purple_zC)
% Column pairs are arranged as X = concentration and Y = penetration distance.
% Exp_* are [z_mm, C_mgL]; model curves use z_mm and C arrays.
series = {
    'Exp_1p8d',       Exp_red_zC(:,2),   Exp_red_zC(:,1);
    'ConstantR_1p8d', C_lin(:,1),        z_mm;
    'Present_1p8d',   C_lan(:,1),        z_mm;
    'Exp_7p5d',       Exp_purple_zC(:,2), Exp_purple_zC(:,1);
    'ConstantR_7p5d', C_lin(:,2),        z_mm;
    'Present_7p5d',   C_lan(:,2),        z_mm;
};

nmax = 0;
for i = 1:size(series,1)
    nmax = max(nmax, numel(series{i,2}));
end

T = table();
for i = 1:size(series,1)
    name = series{i,1};
    C = series{i,2}(:);
    z = series{i,3}(:);
    Cpad = nan(nmax,1); zpad = nan(nmax,1);
    Cpad(1:numel(C)) = C;
    zpad(1:numel(z)) = z;
    T.([name '_C_mgL']) = Cpad;
    T.([name '_z_mm'])  = zpad;
end
end

%% ========================================================================
% Metrics
% ========================================================================

function M = metrics_at_exp(obs, zExp, zModel, cModel)
obs = obs(:);
zExp = zExp(:);
zmin = min(zModel);
zmax = max(zModel);
zUse = min(max(zExp, zmin), zmax);
pred = interp1(zModel, cModel, zUse, 'pchip');
valid = isfinite(obs) & isfinite(pred);
M = metrics(obs(valid), pred(valid));
M.nUsed = sum(valid);
M.nTotal = numel(obs);
end

function M = metrics(obs, pred)
obs = obs(:); pred = pred(:);
M.RMSE = sqrt(mean((pred-obs).^2));
M.MAE  = mean(abs(pred-obs));
den = sum((obs-mean(obs)).^2);
if den > 0
    M.NSE = 1 - sum((obs-pred).^2)/den;
else
    M.NSE = NaN;
end
if std(obs)>0 && std(pred)>0
    M.R2 = corr(obs,pred)^2;
else
    M.R2 = NaN;
end
end


%% ========================================================================
%  FIGURE 5: Two-layer analytical benchmark
% ========================================================================
function make_Fig5_TwoLayer()
fprintf('\n>>> Fig. 5: two-layer benchmark...\n');
secPerYr = 365*24*3600;
interfaces = [0, 0.5, 1.0];
Lphys = diff(interfaces);
n = [0.4, 0.6];
rho = [1500, 1500];
D0 = [5e-9, 1e-8];
aL = [0, 0];
Rd = [2.0, 1.0];
KF = (Rd - 1).*n./rho;   % m3/kg, linear partition coefficient
N = [1, 1];
f = [1, 1];
alpha = [0, 0];
lam = [0, 0];
vDarcy = 4e-9;           % m/s
C0 = 1;
tYears = [0.5, 1, 2, 4, 8];
z = linspace(0,1,161)';

sys = buildSystemFromVelocity(Lphys,n,rho,D0,aL,vDarcy,0.005);
par.N = expandBySystem(sys,N); par.KF = expandBySystem(sys,KF);
par.f = expandBySystem(sys,f); par.alpha = expandBySystem(sys,alpha); par.lam = expandBySystem(sys,lam);

Cmat = zeros(numel(z),numel(tYears));
for k = 1:numel(tYears)
    Cmat(:,k) = solveFreundlich(tYears(k)*secPerYr,z,C0,sys,par.KF,par.N,par.f,par.alpha,par.lam,'Dirichlet','Neumann',0,0);
end
assignin('base','Origin_Fig5_Proposed',[z,Cmat/C0]);

figure('Color','w','Position',[120 80 560 650],'Name','Fig. 5'); hold on; box on;
cols = lines(numel(tYears));
for k = 1:numel(tYears)
    plot(Cmat(:,k)/C0,z,'-','LineWidth',2.2,'Color',cols(k,:), ...
        'DisplayName',sprintf('t = %.1f yr',tYears(k)));
end
yline(0.5,'k--','LineWidth',1.1,'HandleVisibility','off');
text(0.58,0.25,'Layer 1, R_d = 2','FontName','Times New Roman');
text(0.58,0.75,'Layer 2, R_d = 1','FontName','Times New Roman');
set(gca,'YDir','reverse','FontName','Times New Roman','FontSize',12,'LineWidth',1.1,'TickDir','in');
xlabel('Relative concentration, c/c_0','FontName','Times New Roman');
ylabel('Depth, z (m)','FontName','Times New Roman');
xlim([0 1.05]); ylim([0 1]); legend('Location','northeast','Box','off');
title('Fig. 5. Two-layer analytical benchmark','FontName','Times New Roman');
end

%% ========================================================================
%  FIGURE 6: Three-layer source-history benchmark
% ========================================================================
function make_Fig6_ThreeLayer()
fprintf('\n>>> Fig. 6: three-layer benchmark with source decay...\n');
secPerYr = 365*24*3600;
interfaces = [0, 1/3, 2/3, 1.0];
Lphys = diff(interfaces);
n = [0.53, 0.56, 0.37];
rho = [1500, 1500, 1500];
D0 = [4.93e-10, 4.93e-10, 1.95e-10];
aL = [0.004, 0.004, 0.004];
Rd = [1.4, 20, 65];
KF = (Rd - 1).*n./rho;
N = [1,1,1]; f = [1,1,1]; alpha = [0,0,0];
lam = [8.3e-10, 0, 0];
vDarcy = 4.76e-10;
C0 = 1;
tYears = [10,40];
gammaSrc = [0, 0.0365/secPerYr];   % constant source and exponentially decaying source
z = linspace(0,1,201)';

sys = buildSystemFromVelocity(Lphys,n,rho,D0,aL,vDarcy,0.002);
par.N = expandBySystem(sys,N); par.KF = expandBySystem(sys,KF);
par.f = expandBySystem(sys,f); par.alpha = expandBySystem(sys,alpha); par.lam = expandBySystem(sys,lam);

Cmat = zeros(numel(z),4); lab = cell(1,4); ic = 1;
for g = 1:2
    for t = 1:2
        Cmat(:,ic) = solveFreundlich(tYears(t)*secPerYr,z,C0,sys,par.KF,par.N,par.f,par.alpha,par.lam,'Dirichlet','Neumann',gammaSrc(g),0);
        if g==1, sname='Constant source'; else, sname='Decaying source'; end
        lab{ic} = sprintf('%d yr, %s',tYears(t),sname);
        ic = ic + 1;
    end
end
assignin('base','Origin_Fig6_Proposed',[z,Cmat/C0]);

figure('Color','w','Position',[150 80 560 680],'Name','Fig. 6'); hold on; box on;
cols = [0 0.45 0.74; 0 0.45 0.74; 0.85 0.33 0.10; 0.85 0.33 0.10];
sty = {'-','--','-','--'};
for k = 1:4
    plot(Cmat(:,k)/C0,z,sty{k},'Color',cols(k,:),'LineWidth',2.2,'DisplayName',lab{k});
end
yline(1/3,'k--','LineWidth',1.0,'HandleVisibility','off');
yline(2/3,'k--','LineWidth',1.0,'HandleVisibility','off');
set(gca,'YDir','reverse','FontName','Times New Roman','FontSize',12,'LineWidth',1.1,'TickDir','in');
xlabel('Relative concentration, c/c_0','FontName','Times New Roman');
ylabel('Depth, z (m)','FontName','Times New Roman');
xlim([0 1.05]); ylim([0 1]); legend('Location','northeast','Box','off');
title('Fig. 6. Three-layer source-history benchmark','FontName','Times New Roman');
end

%% ========================================================================
%  FIGURE 7: Nonlinear Freundlich + Robin outlet benchmark
% ========================================================================
function make_Fig7_FDM()
fprintf('\n>>> Fig. 7: nonlinear sorption benchmark with Robin outlet...\n');
secPerYr = 365*24*3600;
interfaces = [0,0.60,0.61]; Lphys = diff(interfaces);
n = [0.30,0.70]; rho = [1900,790];
kh = [1.9e-10,5.0e-11];
D0 = [3.78e-10,3.0e-10]; aL = [0.01,0];
KF = [0.63,9.60]*1e-3; N = [0.80,0.98];
f = [1,1]; alpha = [0,0];
lam = [log(2)/(10*secPerYr), log(2)/(10*secPerYr)];
C0List = [5,50,100]; t = 30*secPerYr; z = linspace(0,0.61,400)';
hdiff = 1.0; grad = hdiff/interfaces(end);
muRobin = 0.1;

sys = buildSystemFromKh(Lphys,n,rho,kh,D0,aL,grad,0.001,false);
kbot = muRobin*sys.n(end)*sys.D(end);
par.N = expandBySystem(sys,N); par.KF = expandBySystem(sys,KF);
par.f = expandBySystem(sys,f); par.alpha = expandBySystem(sys,alpha); par.lam = expandBySystem(sys,lam);

Cmat = zeros(numel(z),numel(C0List));
figure('Color','w','Position',[160 90 600 510],'Name','Fig. 7'); hold on; box on;
cols = [0 0.45 0.74; 0.85 0.33 0.10; 0.47 0.67 0.19];
for i = 1:numel(C0List)
    C = solveFreundlich(t,z,C0List(i),sys,par.KF,par.N,par.f,par.alpha,par.lam,'Dirichlet','Robin',0,kbot);
    Cmat(:,i) = C/C0List(i);
    plot(z,Cmat(:,i),'-','LineWidth',2.3,'Color',cols(i,:),'DisplayName',sprintf('c_0 = %d mg/L',C0List(i)));
end
assignin('base','Origin_Fig7_Proposed',[z,Cmat]);
xline(0.60,'k--','LineWidth',1.2,'HandleVisibility','off');
patch([0 0.60 0.60 0],[0 0 1.05 1.05],[0.96 0.96 0.96],'EdgeColor','none','FaceAlpha',0.4,'HandleVisibility','off');
uistack(findobj(gca,'Type','Patch'),'bottom');
text(0.30,0.14,'SB','HorizontalAlignment','center','FontName','Times New Roman');
text(0.605,0.55,'GCL','Rotation',90,'HorizontalAlignment','center','FontName','Times New Roman');
set(gca,'FontName','Times New Roman','FontSize',12,'LineWidth',1.1,'TickDir','in');
xlabel('Distance, z (m)','FontName','Times New Roman'); ylabel('Relative concentration, c/c_0','FontName','Times New Roman');
xlim([0 0.61]); ylim([0 1.05]); legend('Location','northeast','Box','off');
title('Fig. 7. Nonlinear sorption benchmark','FontName','Times New Roman');
end

%% ========================================================================
%  FIGURE 8: FEM verification scenarios without benchmark symbols
% ========================================================================
function make_Fig8_FEM()
fprintf('\n>>> Fig. 8: Freundlich and Langmuir FEM verification scenarios...\n');
secPerYr = 365*24*3600;
[sys, base] = standardCutoffSystem(0.05);
C0List = [1,10,50,100]; x = linspace(0,12,25)'; t = 100*secPerYr;
cols = [0 0 0; 0 0.30 0.85; 0.85 0.10 0.10; 0 0.55 0.20];
sty = {'-','--','-.',':'};

figure('Color','w','Position',[80 60 1120 820],'Name','Fig. 8');
allData = struct();
for row = 1:2
    if row==1
        sorp = 'Freundlich';
    else
        sorp = 'Langmuir';
    end
    for col = 1:2
        if col==1, bcIn='Dirichlet'; else, bcIn='Flux'; end
        subplot(2,2,(row-1)*2+col); hold on; box on;
        shadeWall(gca,0,1.05);
        data = x;
        for i = 1:numel(C0List)
            C0 = C0List(i);
            if strcmpi(sorp,'Freundlich')
                C = solveFreundlich(t,x,C0,sys,base.KF,base.N,base.f,base.alpha,base.lam,bcIn,'Neumann',0,0);
            else
                C = solveLangmuir(t,x,C0,sys,base.Qm,base.KL,base.f,base.alpha,base.lam,bcIn,'Neumann',0,0);
            end
            plot(x,C/C0,sty{i},'Color',cols(i,:),'LineWidth',2.1,'DisplayName',sprintf('c_0 = %d mg/L',C0));
            data = [data,C/C0]; %#ok<AGROW>
        end
        set(gca,'FontName','Times New Roman','FontSize',11,'LineWidth',1.1,'TickDir','in');
        xlabel('Distance, x (m)','FontName','Times New Roman'); ylabel('Relative concentration, c/c_0','FontName','Times New Roman');
        xlim([0 12]); ylim([0 1.05]);
        title(sprintf('(%s) %s, %s-Neumann',char(96+(row-1)*2+col),sorp,bcIn),'FontName','Times New Roman');
        if row==1 && col==1, legend('Location','northeast','Box','off'); end
        allData.(sprintf('Fig8_%s_%s',sorp,bcIn)) = data;
    end
end
assignin('base','Origin_Fig8_Proposed',allData);
end

%% ========================================================================
%  FIGURE 9: Mechanistic comparison NENL/ENL/NEL/EL
% ========================================================================
function make_Fig9_Mechanistic()
fprintf('\n>>> Fig. 9: NENL/ENL/NEL/EL comparison...\n');
secPerYr = 365*24*3600;
[sys, base] = standardCutoffSystem(0.05);
C0 = 100;
models = defineMechanisticModels(base);
modelNames = {'EL','NEL','ENL','NENL'};
cols = [0 0 0; 0.85 0.10 0.10; 0 0.30 0.85; 0 0.55 0.20];
sty = {':','--','-.','-'};

% Fig. 9a: spatial profile at 100 years
x = linspace(0,3.5,220)'; tProfile = 100*secPerYr;
Cprof = zeros(numel(x),4);
for i = 1:4
    Cprof(:,i) = solveFreundlich(tProfile,x,C0,sys,base.KF,models(i).N,models(i).f,models(i).alpha,base.lam,'Dirichlet','Neumann',0,0)/C0;
end
assignin('base','Origin_Fig9a_Profile',[x,Cprof]);
figure('Color','w','Position',[90 70 1080 800],'Name','Fig. 9');
subplot(2,2,1); hold on; box on; shadeWall(gca,0,1.05);
for i=1:4, plot(x,Cprof(:,i),sty{i},'Color',cols(i,:),'LineWidth',2.2,'DisplayName',modelNames{i}); end
setFigAxes('Position, x (m)','Relative concentration, c/c_0'); xlim([0 3.5]); ylim([0 1.05]);
title('(a) Spatial concentration profiles','FontName','Times New Roman'); legend('Location','northeast','Box','off');

% Fig. 9b-c: BTCs at x = 1 and x = 2
xObs = [1.0,2.0]; tYears = logspace(0,3,70); BTC = zeros(numel(tYears),4,2);
for p = 1:2
    for it = 1:numel(tYears)
        tt = tYears(it)*secPerYr;
        for i = 1:4
            BTC(it,i,p) = solveFreundlich(tt,xObs(p),C0,sys,base.KF,models(i).N,models(i).f,models(i).alpha,base.lam,'Dirichlet','Neumann',0,0)/C0;
        end
    end
end
assignin('base','Origin_Fig9b_BTC_x1',[tYears',BTC(:,:,1)]);
assignin('base','Origin_Fig9c_BTC_x2',[tYears',BTC(:,:,2)]);
for p=1:2
    subplot(2,2,1+p); hold on; box on;
    for i=1:4, plot(tYears,BTC(:,i,p),sty{i},'Color',cols(i,:),'LineWidth',2.1,'DisplayName',modelNames{i}); end
    set(gca,'XScale','log'); setFigAxes('Time, t (year)','Relative concentration, c/c_0');
    xlim([1 1000]); ylim([0 1.05]);
    if p==1, title('(b) Breakthrough curves at x = 1 m','FontName','Times New Roman'); else, title('(c) Breakthrough curves at x = 2 m','FontName','Times New Roman'); end
end

% Fig. 9d: relative error between EL and NENL
xg = linspace(0,3.5,80); tg = logspace(log10(1),log10(1000),60); Err = nan(numel(tg),numel(xg));
for it = 1:numel(tg)
    tt = tg(it)*secPerYr;
    C_EL   = solveFreundlich(tt,xg,C0,sys,base.KF,models(1).N,models(1).f,models(1).alpha,base.lam,'Dirichlet','Neumann',0,0)/C0;
    C_NENL = solveFreundlich(tt,xg,C0,sys,base.KF,models(4).N,models(4).f,models(4).alpha,base.lam,'Dirichlet','Neumann',0,0)/C0;
    e = abs(C_EL - C_NENL)./max(C_NENL,1e-8)*100;
    e(C_NENL<1e-5) = NaN;
    Err(it,:) = e;
end
assignin('base','Origin_Fig9d_Error_XYZ',xyzFromGrid(xg,tg,Err));
subplot(2,2,4); hold on; box on;
contourf(xg,tg,Err,18,'LineColor','none'); colorbar; set(gca,'YScale','log');
xline(1,'k--','LineWidth',1.0); xline(2,'k--','LineWidth',1.0);
setFigAxes('Position, x (m)','Time, t (year)'); title('(d) Relative error |c_{EL}-c_{NENL}|/c_{NENL}','FontName','Times New Roman');
end

%% ========================================================================
%  FIGURE 10: Non-equilibrium parameter effects
% ========================================================================
function make_Fig10_NEParams()
fprintf('\n>>> Fig. 10: non-equilibrium parameter effects...\n');
secPerYr = 365*24*3600;
[sys, base] = standardCutoffSystem(0.05);
C0=100; xObs=2.0; threshold=0.1; tYears=logspace(0,3,90);
ks_day = [Inf,1e-1,1e-4,1e-7];
fixedF = 0.5;
fCases = [1.0,0.9,0.5,0.1];
fixedKs = 1e-4;
cols = [0 0 0; 0 0.30 0.85; 0.85 0.10 0.10; 0 0.55 0.20];
sty = {'-','--','-.',':'};

BTC_ks = zeros(numel(tYears),numel(ks_day));
for j=1:numel(ks_day)
    if isinf(ks_day(j)), aLayer = [1e6,1e6,1e6]/86400; else, aLayer = [ks_day(j),ks_day(j),ks_day(j)]/86400; end
    fLayer = [fixedF,fixedF,fixedF];
    alpha = expandBySystem(sys,aLayer); f = expandBySystem(sys,fLayer);
    for it=1:numel(tYears)
        BTC_ks(it,j) = solveFreundlich(tYears(it)*secPerYr,xObs,C0,sys,base.KF,base.N,f,alpha,base.lam,'Dirichlet','Neumann',0,0)/C0;
    end
end
BT_ks = breakthroughTimes(tYears,BTC_ks,threshold);

BTC_f = zeros(numel(tYears),numel(fCases));
alphaFixed = expandBySystem(sys,[fixedKs,fixedKs,fixedKs]/86400);
for j=1:numel(fCases)
    f = expandBySystem(sys,[fCases(j),fCases(j),fCases(j)]);
    for it=1:numel(tYears)
        BTC_f(it,j) = solveFreundlich(tYears(it)*secPerYr,xObs,C0,sys,base.KF,base.N,f,alphaFixed,base.lam,'Dirichlet','Neumann',0,0)/C0;
    end
end
BT_f = breakthroughTimes(tYears,BTC_f,threshold);
assignin('base','Origin_Fig10a_ks_BTC',[tYears',BTC_ks]);
assignin('base','Origin_Fig10b_f_BTC',[tYears',BTC_f]);
assignin('base','Origin_Fig10c_BT',{ks_day,BT_ks,fCases,BT_f});

figure('Color','w','Position',[70 110 1280 390],'Name','Fig. 10');
subplot(1,3,1); hold on; box on;
for j=1:numel(ks_day)
    if isinf(ks_day(j)), lab='k_s \rightarrow \infty'; else, lab=sprintf('k_s = 10^{%d} d^{-1}',round(log10(ks_day(j)))); end
    plot(tYears,BTC_ks(:,j),sty{j},'Color',cols(j,:),'LineWidth',2.1,'DisplayName',lab);
end
yline(threshold,'k:','c/c_0=0.1','HandleVisibility','off'); set(gca,'XScale','log');
setFigAxes('Time, t (year)','Relative concentration, c/c_0'); xlim([1 1000]); ylim([1e-4 1.05]); title('(a) Effect of k_s','FontName','Times New Roman'); legend('Location','southeast','Box','off');

subplot(1,3,2); hold on; box on;
for j=1:numel(fCases)
    plot(tYears,BTC_f(:,j),sty{j},'Color',cols(j,:),'LineWidth',2.1,'DisplayName',sprintf('f_i = %.1f',fCases(j)));
end
yline(threshold,'k:','c/c_0=0.1','HandleVisibility','off'); set(gca,'XScale','log');
setFigAxes('Time, t (year)','Relative concentration, c/c_0'); xlim([1 1000]); ylim([1e-4 1.05]); title('(b) Effect of f_i','FontName','Times New Roman'); legend('Location','southeast','Box','off');

subplot(1,3,3); hold on; box on;
ksPlot = [1e-1,1e-4,1e-7]; btPlot = BT_ks(2:end);
semilogx(ksPlot,btPlot,'s-','LineWidth',2.1,'MarkerFaceColor','w','DisplayName','k_s');
plot(fCases,BT_f,'o-','LineWidth',2.1,'MarkerFaceColor','w','DisplayName','f_i');
set(gca,'XDir','reverse'); setFigAxes('Parameter value','Breakthrough time (year)');
title('(c) Breakthrough-time relationships','FontName','Times New Roman'); legend('Location','best','Box','off');
end

%% ========================================================================
%  FIGURE 11: Freundlich exponent and apparent retardation
% ========================================================================
function make_Fig11_FreundlichExponent()
fprintf('\n>>> Fig. 11: nonlinear Freundlich exponent effects...\n');
secPerYr = 365*24*3600;
[sys, base0] = standardCutoffSystem(0.05);
C0=100; xObs=2.0; Ncases=[1.0,0.85,0.70];
f = expandBySystem(sys,[0.5,0.5,0.5]);
alpha = expandBySystem(sys,[1e-4,1e-4,1e-4]/86400);
tYears=logspace(0,log10(300),80); x=linspace(0,3.5,200)'; tProfile=100*secPerYr;
cols=[0 0 0;0 0.30 0.85;0.85 0.10 0.10]; sty={'-','--','-.'};
BTC=zeros(numel(tYears),numel(Ncases)); Rprof=zeros(numel(x),numel(Ncases));

for j=1:numel(Ncases)
    N = expandBySystem(sys,[Ncases(j),Ncases(j),Ncases(j)]);
    for it=1:numel(tYears)
        BTC(it,j)=solveFreundlich(tYears(it)*secPerYr,xObs,C0,sys,base0.KF,N,f,alpha,base0.lam,'Dirichlet','Neumann',0,0)/C0;
    end
    Csp = solveFreundlich(tProfile,x,C0,sys,base0.KF,N,f,alpha,base0.lam,'Dirichlet','Neumann',0,0);
    Rprof(:,j)=apparentR_Freundlich(x,Csp,Ncases(j),sys,base0);
end
assignin('base','Origin_Fig11a_BTC',[tYears',BTC]);
assignin('base','Origin_Fig11b_Rstar',[x,Rprof]);

figure('Color','w','Position',[60 110 1280 390],'Name','Fig. 11');
subplot(1,3,1); hold on; box on;
for j=1:numel(Ncases)
    plot(tYears,BTC(:,j),sty{j},'Color',cols(j,:),'LineWidth',2.2,'DisplayName',sprintf('N_i = %.2f',Ncases(j)));
end
yline(0.1,'k:','c/c_0=0.1','HandleVisibility','off'); set(gca,'XScale','log');
setFigAxes('Time, t (year)','Relative concentration, c/c_0'); xlim([1 300]); ylim([1e-4 1.05]); title('(a) Breakthrough at x = 2 m','FontName','Times New Roman'); legend('Location','southeast','Box','off');

subplot(1,3,2); hold on; box on; shadeWall(gca,0,max(Rprof(:))*1.05);
for j=1:numel(Ncases)
    plot(x,Rprof(:,j),sty{j},'Color',cols(j,:),'LineWidth',2.2,'DisplayName',sprintf('N_i = %.2f',Ncases(j)));
end
setFigAxes('Position, x (m)','Apparent retardation factor, R_d^*'); xlim([0 3.5]); ylim([0 max(Rprof(:))*1.05]); title('(b) R_d^* at t = 100 years','FontName','Times New Roman');

% Fig. 11c: 10-100 years inside the cutoff wall, according to manuscript comment
xg=linspace(1,2,80); tg=logspace(1,2,55); Err=nan(numel(tg),numel(xg));
Nlin = expandBySystem(sys,[1,1,1]); Nnon = expandBySystem(sys,[0.7,0.7,0.7]);
for it=1:numel(tg)
    Cn = solveFreundlich(tg(it)*secPerYr,xg,C0,sys,base0.KF,Nnon,f,alpha,base0.lam,'Dirichlet','Neumann',0,0);
    Rlin = apparentR_Freundlich(xg,Cn,1.0,sys,base0);
    Rnon = apparentR_Freundlich(xg,Cn,0.7,sys,base0);
    Err(it,:)=(Rlin-Rnon)./max(Rnon,1e-12)*100;
end
assignin('base','Origin_Fig11c_Rerror_XYZ',xyzFromGrid(xg,tg,Err));
subplot(1,3,3); hold on; box on;
contourf(xg,tg,Err,18,'LineColor','none'); colorbar; set(gca,'YScale','log');
setFigAxes('Position in cutoff wall, x (m)','Time, t (year)'); title('(c) Error in R_d^*, N=1 vs N=0.7','FontName','Times New Roman');
end

%% ========================================================================
%  SHARED MODEL/SOLVER FUNCTIONS
% ========================================================================
function [sys, base] = standardCutoffSystem(dz)
secPerYr = 365*24*3600;
Lphys=[1,1,10]; n=[0.55,0.50,0.55]; rho=[1190,1340,1190]; kh=[1e-6,1e-9,1e-6];
tau=[0.45,0.40,0.45]; Dw=[9.11e-10,6.53e-10,1.07e-9]; aL=[0.44,0.20,0.6183];
hdiff=1.0; grad=hdiff/sum(Lphys);
Dbase = tau.*Dw;  % molecular part after tortuosity
sys = buildSystemFromKh(Lphys,n,rho,kh,Dbase,aL,grad,dz,true);
base.KF = expandBySystem(sys,[0.429,4.45,0.416]*1e-3);
base.N  = expandBySystem(sys,[0.80,0.665,0.80]);
base.f  = expandBySystem(sys,[0.40,0.60,0.40]);
base.alpha = expandBySystem(sys,[1e-4,1e-4,1e-4]/86400);
base.lam = expandBySystem(sys,[log(2)/(50*secPerYr),log(2)/(50*secPerYr),log(2)/(50*secPerYr)]);
% Langmuir values are illustrative FEM verification settings; units give m3/kg naturally.
base.Qm = expandBySystem(sys,[0.005,0.020,0.005]);
base.KL = expandBySystem(sys,[0.010,0.050,0.010]);
base.layerL=Lphys; base.layerN=n; base.layerRho=rho; base.layerKF=[0.429,4.45,0.416]*1e-3;
end

function sys = buildSystemFromKh(Lphys,n,rho,kh,D0,aL,grad,dz,khHarmonic)
if nargin<9, khHarmonic=true; end
if khHarmonic
    Keq = sum(Lphys)/sum(Lphys./kh);
    vDarcy = Keq*grad;
else
    Keq = sum(Lphys)/sum(Lphys./kh);
    vDarcy = Keq*grad;
end
sys = buildSystemFromVelocity(Lphys,n,rho,D0,aL,vDarcy,dz);
end

function sys = buildSystemFromVelocity(Lphys,n,rho,D0,aL,vDarcy,dz)
L=[]; nv=[]; rhov=[]; vv=[]; Dv=[]; layerId=[];
for i=1:numel(Lphys)
    m=ceil(Lphys(i)/dz); dzi=Lphys(i)/m; oi=ones(1,m);
    vp=vDarcy/n(i);
    D=D0(i)+aL(i)*vp;
    L=[L,oi*dzi]; nv=[nv,oi*n(i)]; rhov=[rhov,oi*rho(i)]; vv=[vv,oi*vp]; Dv=[Dv,oi*D]; layerId=[layerId,oi*i]; %#ok<AGROW>
end
sys.L=L; sys.n=nv; sys.rho=rhov; sys.v=vv; sys.D=Dv; sys.layerId=layerId; sys.edges=[0,cumsum(L)];
end

function vec = expandBySystem(sys,layerVals)
vec = zeros(size(sys.L));
for i=1:max(sys.layerId)
    vec(sys.layerId==i)=layerVals(i);
end
end

function models = defineMechanisticModels(base)
% EL: linear equilibrium; NEL: linear non-equilibrium; ENL: nonlinear equilibrium; NENL: full model
models(1).N = ones(size(base.N)); models(1).f = ones(size(base.f)); models(1).alpha = base.alpha;
models(2).N = ones(size(base.N)); models(2).f = base.f; models(2).alpha = base.alpha;
models(3).N = base.N; models(3).f = ones(size(base.f)); models(3).alpha = base.alpha;
models(4).N = base.N; models(4).f = base.f; models(4).alpha = base.alpha;
end

function C = solveFreundlich(t,xEval,C0,sys,KF,N,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot)
% Homotopy-Picard with tangent/secant separation for Freundlich sorption.
if max(abs(N-1))<1e-12
    Ksec = KF; Ktan = KF;
    C = computeLaplace(t,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
    C = max(real(C),0); return;
end
xMid = sys.edges(1:end-1)+sys.L/2;
pSteps = linspace(0,1,5);
Ksec = KF; Ktan = KF;
Ccur = computeLaplace(t,xMid,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
Ccur = max(real(Ccur),0);
for h=1:numel(pSteps)
    Neff = 1 + pSteps(h)*(N-1);
    for iter=1:80
        Cs = max(Ccur,1e-12);
        KsecTarget = KF.*(Cs.^(Neff-1));
        KtanTarget = Neff.*KsecTarget;
        w = 0.55;
        Ksec = (1-w)*Ksec + w*KsecTarget;
        Ktan = (1-w)*Ktan + w*KtanTarget;
        Cnew = computeLaplace(t,xMid,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
        Cnew = max(real(Cnew),0);
        err = max(abs(Cnew-Ccur)./(abs(Cnew)+1e-10));
        Ccur = Cnew;
        if err < 1e-4, break; end
    end
end
C = computeLaplace(t,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
C = max(real(C),0);
end

function C = solveLangmuir(t,xEval,C0,sys,Qm,KL,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot)
xMid = sys.edges(1:end-1)+sys.L/2;
pSteps = linspace(0,1,5);
Ksec = Qm.*KL; Ktan = Qm.*KL;
Ccur = computeLaplace(t,xMid,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
Ccur = max(real(Ccur),0);
for h=1:numel(pSteps)
    g = pSteps(h);
    for iter=1:80
        Cs = max(Ccur,1e-12);
        denom = 1 + g.*KL.*Cs;
        KsecTarget = (Qm.*KL)./denom;
        KtanTarget = (Qm.*KL)./(denom.^2);
        w=0.55;
        Ksec=(1-w)*Ksec+w*KsecTarget;
        Ktan=(1-w)*Ktan+w*KtanTarget;
        Cnew = computeLaplace(t,xMid,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
        Cnew = max(real(Cnew),0);
        err=max(abs(Cnew-Ccur)./(abs(Cnew)+1e-10));
        Ccur=Cnew;
        if err<1e-4, break; end
    end
end
C=computeLaplace(t,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
C=max(real(C),0);
end

function C = computeLaplace(t,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot)
if t<=0, C=zeros(size(xEval)); return; end
M=40; val=zeros(size(xEval));
for k=0:M-1
    if k==0
        s = 2*M/5/t;
        gam = 0.5*exp(s*t);
    else
        theta=k*pi/M;
        s = 2*M/5/t*(theta*cot(theta)+1i*theta);
        sigma = theta + (theta*cot(theta)-1)*cot(theta);
        gam = exp(s*t)*(1+1i*sigma);
    end
    Cs = solveMatrix(s,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot);
    val = val + gam.*Cs;
end
C = 2/(5*t)*real(val);
end

function val = solveMatrix(s,xEval,C0,sys,Ksec,Ktan,f,alpha,lam,bcIn,bcOut,gammaSrc,kbot)
L=sys.L; n=sys.n; rho=sys.rho; D=sys.D; v=sys.v; m=numel(L);
acc = s.*(1 + (rho./n).*f.*Ktan);
dec = lam.*(1 + (rho./n).*f.*Ksec);
kin = (rho./n).*(alpha.*(1-f).*(s+lam).*Ksec)./(s+alpha+lam+eps);
Psi = acc + dec + kin;
delta = sqrt(v.^2 + 4.*D.*Psi);
Lam1 = (v + delta)./(2.*D);
Lam2 = (v - delta)./(2.*D);
Kap = n.*D;
Nunk=2*m; A=spalloc(Nunk,Nunk,4*Nunk); b=zeros(Nunk,1);
C0s = C0/(s+gammaSrc);

% inlet boundary
if strcmpi(bcIn,'Flux')
    A(1,1)=exp(-Lam1(1)*L(1))*(v(1)-D(1)*Lam1(1));
    A(1,2)=v(1)-D(1)*Lam2(1);
    b(1)=v(1)*C0s;
else
    A(1,1)=exp(-Lam1(1)*L(1)); A(1,2)=1; b(1)=C0s;
end

% interface continuity
for i=1:m-1
    r=2*i; idx=2*i-1;
    E2=exp(Lam2(i)*L(i)); E1n=exp(-Lam1(i+1)*L(i+1));
    A(r,idx)=1; A(r,idx+1)=E2; A(r,idx+2)=-E1n; A(r,idx+3)=-1;
    A(r+1,idx)=Kap(i)*Lam1(i); A(r+1,idx+1)=Kap(i)*Lam2(i)*E2;
    A(r+1,idx+2)=-Kap(i+1)*Lam1(i+1)*E1n; A(r+1,idx+3)=-Kap(i+1)*Lam2(i+1);
end

% outlet boundary
Em=exp(Lam2(m)*L(m));
if strcmpi(bcOut,'Robin')
    A(Nunk,Nunk-1)=Kap(m)*Lam1(m)+kbot;
    A(Nunk,Nunk)=(Kap(m)*Lam2(m)+kbot)*Em;
else
    A(Nunk,Nunk-1)=Lam1(m);
    A(Nunk,Nunk)=Lam2(m)*Em;
end
coef=A\b;
edges=sys.edges; val=zeros(size(xEval));
for q=1:numel(xEval)
    x=xEval(q);
    id=find(x<=edges(2:end)+1e-12,1,'first'); if isempty(id), id=m; end
    xloc=x-edges(id);
    val(q)=coef(2*id-1)*exp(-Lam1(id)*(L(id)-xloc)) + coef(2*id)*exp(Lam2(id)*xloc);
end
end

%% ========================================================================
%  UTILITY FUNCTIONS
% ========================================================================
function shadeWall(ax,ymin,ymax)
axes(ax); %#ok<LAXES>
patch([1 2 2 1],[ymin ymin ymax ymax],[0.94 0.94 0.94], ...
    'EdgeColor','none','FaceAlpha',0.55,'HandleVisibility','off');
xline(1,'k--','LineWidth',0.9,'HandleVisibility','off');
xline(2,'k--','LineWidth',0.9,'HandleVisibility','off');
uistack(findobj(ax,'Type','Patch'),'bottom');
end

function setFigAxes(xlab,ylab)
set(gca,'FontName','Times New Roman','FontSize',11,'LineWidth',1.1,'TickDir','in');
xlabel(xlab,'FontName','Times New Roman'); ylabel(ylab,'FontName','Times New Roman');
end

function XYZ = xyzFromGrid(x,y,Z)
[X,Y]=meshgrid(x,y); XYZ=[X(:),Y(:),Z(:)];
end

function bt = breakthroughTimes(tYears,BTC,thr)
bt = nan(1,size(BTC,2));
for j=1:size(BTC,2)
    y=BTC(:,j);
    idx=find(y>=thr,1,'first');
    if isempty(idx)
        bt(j)=NaN;
    elseif idx==1
        bt(j)=tYears(1);
    else
        bt(j)=interp1(y(idx-1:idx),tYears(idx-1:idx),thr,'linear','extrap');
    end
end
end

function R = apparentR_Freundlich(x,C,Nval,sys,base)
R=zeros(size(x));
for i=1:numel(x)
    lid=find(x(i)<=cumsum(base.layerL)+1e-12,1,'first'); if isempty(lid), lid=numel(base.layerL); end
    Csafe=max(C(i),1e-12);
    R(i)=1+(base.layerRho(lid)/base.layerN(lid))*base.layerKF(lid)*(Csafe.^(Nval-1));
end
end
